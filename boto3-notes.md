# Boto3 Notes
Boto3 is the Amazon Web Services (AWS) Software Development Kit (SDK) for Python, which allows developers to write applications that can interact with AWS services.

## Client Vs Resource
- **Client**: Provides a low-level interface to AWS services, allowing you to make direct API calls.
- **Resource**: Provides a higher-level, object-oriented interface to AWS services, making it easier to work with AWS resources. But it has the following:
  - **Hidden Performance Costs**: High-level abstractions can hide network calls. Accessing a property on an object might silently trigger an extra API roundtrip, causing unexpected latency and hitting API rate limits.

- **Limitation in Resource**:
  - Feature Freeze: In early 2023, AWS announced that the resources interface is under an official feature freeze.
  - No New Updates: AWS will not add new features, support new services, or accept resource model changes.
  - Legacy Support Only: Existing resource methods in boto3 will continue to work for backwards compatibility throughout the Boto3 lifecycle, but won't be carried forward to any future major version of the AWS SDK for Python.
  - The Industry Standard: boto3.client is the permanent default for Python development on AWS. If teams want high-level abstractions today, they use Infrastructure-as-Code (Terraform, AWS CDK) or third-party ORMs/frameworks rather than boto3.resource.

Example of creating a vpc and subnet with Client and Resource:

```python
import boto3

# Using Client
ec2_client = boto3.client('ec2')
vpc = ec2_client.create_vpc(CidrBlock='10.0.0.0/16')
subnet = ec2_client.create_subnet(VpcId=vpc['Vpc']['VpcId'], CidrBlock='10.0.1.0/24')

# Using Resource
ec2_resource = boto3.resource('ec2')
vpc_resource = ec2_resource.create_vpc(CidrBlock='10.0.0.0/16')
subnet_resource = ec2_resource.create_subnet(VpcId=vpc_resource.id, CidrBlock='10.0.1.0/24')
```

## Boto3 Vs Terraform

### 1. Programming Paradigm
- **Boto3 (Imperative):** You tell AWS *how* to do something step-by-step (`create_vpc()`, `run_instances()`).
- **Terraform (Declarative):** You declare the *desired end-state* in HashiCorp Configuration Language (HCL), and Terraform computes the diff to reach that state.

### 2. State Management
- **Boto3:** Completely stateless. It does not remember past runs; you must manually write logic to inspect whether a resource already exists or needs updating.
- **Terraform:** State-driven. It tracks real-world infrastructure using a state file (`terraform.tfstate`), making operations idempotent.

### 3. Idempotency
- **Boto3:** Not idempotent by default. Running a creation script twice will typically create duplicate resources or throw an error unless explicitly coded around.
- **Terraform:** Native idempotency. Running `terraform apply` repeatedly without code changes results in zero modifications.

### 4. Dependency Resolution
- **Boto3:** Manual. You must handle the order of operations, polling, and waits yourself (e.g., waiting for a VPC or Subnet to become available before launching an EC2 instance).
- **Terraform:** Automatic. It constructs an internal Dependency Graph and provisions resources concurrently or sequentially based on variable references.

### 5. Cloud Scope
- **Boto3:** AWS-only.
- **Terraform:** Multi-cloud and multi-provider (AWS, GCP, Azure, Kubernetes, GitHub, Datadog, etc.) using a unified workflow.

### 6. Drift Detection & Lifecycle Management
- **Boto3:** No built-in drift detection or lifecycle commands. You must write custom scripts to check or delete what was built.
- **Terraform:** Built-in drift detection (`terraform plan`) and clean teardown (`terraform destroy`).

### 7. Typical Production Use Cases

- With boto3 we can write more complex logic and workflows which we can't in Terraform.
- Application runtime code (e.g., an app uploading files to S3 or reading DynamoDB).
- Monitoring, Backups, Scheduled Tasks etc.
- Serverless AWS Lambda execution logic.
- Dynamic, event-driven tasks (e.g., stopping non-prod EC2 instances nightly, rotating API keys, custom security auditing scripts).

- **Use Terraform for**:
- Base and persistent infrastructure provisioning (VPCs, subnets, IAM policies, EKS clusters, databases, routing tables).
- Managing environments across dev, staging, and production consistently.
