# Kubernetes – Deep Short Notes

- **Playlist**: Docker and Kubernetes Tutorial for Beginners – TechWorld with Nana [Link](https://youtube.com/playlist?list=PLy7NrYWoggjwPggqtFsI_zMAwvG0SqYCb&si=lBQuMIQ0CFECW_K5)
- Total Notes credits goes to TechWorld with Nana for the original content and explanations in the videos.

---

## 1. Kubernetes Building Blocks (Pods, Services, ConfigMaps, Secrets)

- Cluster as a collection of nodes: A Kubernetes cluster is a group of worker machines (nodes) that run containers and a control plane that manages what runs where.  
- Pod as smallest deployable unit: A Pod wraps one or more tightly‑coupled containers that share IP, ports, and storage, representing a single deployable “instance” of an application.  
- One app instance per Pod: Typically each Pod runs one main application container (plus optional sidecars) so scaling the Deployment creates more Pod replicas, each running that app.  
- Container sharing inside a Pod: Containers in the same Pod share the same network namespace and can reach each other via `localhost`, making them behave like processes in the same host.  
- Service as stable access point: A Service exposes a set of Pods under a stable virtual IP and DNS name, so clients do not need to track Pod IPs that change over time.  
- Label selector concept: Services choose which Pods to route to by matching labels, decoupling the client address from the specific Pods behind it.  
- ConfigMap for non‑secret config: ConfigMaps hold configuration data (e.g., URLs, feature flags) as key‑value pairs that can be injected into Pods as environment variables or files.  
- Secret for sensitive data: Secrets store confidential information (passwords, tokens, certificates) in a separate resource with access control and are mounted or injected similar to ConfigMaps.  
- Declarative objects via YAML: Pods, Services, ConfigMaps, and Secrets are defined as YAML manifests, applied to the cluster so Kubernetes continuously keeps actual state in line with the declared spec.  

---

## 2. Kubernetes Architecture

- Control plane responsibility: The control plane maintains the desired state of the cluster, deciding which workloads run where and reacting to failures.  
- API server as front door: The API server is the central HTTP endpoint that all tools and internal components talk to, validating requests and persisting changes.  
- etcd as state store: etcd is a distributed key‑value store that holds the entire cluster configuration and state, including objects like Pods, Services, and Deployments.  
- Controller manager: A set of controllers monitors desired vs actual state (e.g., replica counts, node health) and makes changes to move the system toward the declared configuration.  
- Scheduler: The scheduler watches unscheduled Pods and assigns them to nodes based on resource availability, taints/tolerations, and other constraints.  
- Worker node components: Each node runs a kubelet, kube‑proxy, and container runtime to manage Pods, handle networking, and enforce Pod specifications locally.  
- Kubelet’s reconciliation: Kubelet regularly checks which Pods should be on its node and ensures their containers are running with the correct image, configuration, and health status.  
- Kube‑proxy and Service routing: Kube‑proxy programs low‑level networking rules (iptables or IPVS) on nodes so traffic to a Service IP gets load‑balanced to one of the backing Pod endpoints.  

---

## 3. Why Kubernetes? (Scalability, High Availability, Disaster Recovery)

- Declarative scaling: Deployments specify a replica count, and Kubernetes ensures that number of Pod instances exist, enabling easy horizontal scaling by just changing a single number.  
- High availability via replicas: By running multiple replicas of an app across nodes, Kubernetes keeps services available even if some Pods or nodes fail.  
- Self‑healing behavior: Kubernetes automatically restarts crashed containers, recreates failed Pods, and reschedules workloads if nodes go down, reducing manual ops work.  
- Rolling updates: Deployments support rolling updates, gradually replacing old Pods with new ones while keeping the app serving traffic, minimizing downtime.  
- Rollbacks: If a new version causes issues, Kubernetes can roll back a Deployment to a previously known good ReplicaSet with a single command or config change.  
- Cluster‑level resource optimization: The scheduler packs Pods across nodes according to resource requests, making better use of cluster CPU and memory.  
- Disaster recovery patterns: Having workloads defined declaratively in manifests and images in registries allows clusters to be recreated in another region or environment when needed.  

---

## 4. Local Setup: Minikube & kubectl

- Minikube as local K8s: Minikube runs a single‑node Kubernetes cluster on your laptop using a VM or Docker driver, ideal for learning and development.  
- Drivers and backends: Minikube can use different virtualization drivers (Docker, Hyper‑V, VirtualBox, etc.), and you must pick one compatible with your OS setup.  
- Starting a cluster: `minikube start` provisions the node, installs Kubernetes components, and configures your local kubeconfig to point `kubectl` at the new cluster.  
- Kubectl as CLI client: `kubectl` is the main CLI for interacting with Kubernetes, sending requests to the API server using the contexts and credentials in `~/.kube/config`.  
- Contexts and clusters: The kubeconfig can hold references to multiple clusters, and contexts define which cluster and namespace kubectl commands target by default.  
- Basic health checks: After setup, commands like `kubectl get nodes` and `kubectl get pods -A` confirm that the cluster is running and system Pods are healthy.  

---

## 5. kubectl Basics (Create & Debug)

- `kubectl apply -f file.yaml`: Applies or updates the resources described in the YAML manifest, making it the standard way to introduce declarative changes.  
- `kubectl get <resource>`: Lists resources like Pods, Services, and Deployments, and with `-o wide` provides extra details such as node and Pod IP.  
- `kubectl describe <resource/name>`: Shows extensive information about a specific resource, including events, which is useful to diagnose scheduling and image pull problems.  
- `kubectl logs pod-name`: Fetches logs from a container in a Pod; with `-c` you can select a particular container when there are multiple in one Pod.  
- `kubectl logs -f`: Follows log output in real time, mirroring `docker logs -f`, which is helpful while interacting with an application.  
- `kubectl exec -it pod-name -- sh/bash`: Opens an interactive shell inside a running container, allowing inspection of environment, filesystem, and network from the Pod’s perspective.  
- `kubectl delete -f file.yaml` or `kubectl delete pod/name`: Removes resources, allowing you to clean up and redeploy during experiments.  

---

## 6. Kubernetes YAML: Deployments & Services

- apiVersion and kind fields: Each YAML defines what type of Kubernetes object it is (e.g., `Deployment`, `Service`) and which API version it belongs to, impacting available fields.  
- Metadata section: `metadata` contains the resource name, namespace, and labels/annotations that identify and categorize objects for selection and tooling.  
- Deployment spec and replicas: A Deployment’s `spec.replicas` field declares how many Pod instances should run, and Kubernetes ensures that count is maintained.  
- Pod template under Deployment: `spec.template` describes the Pod: containers, images, ports, environment variables, probes, and volume mounts for each replica managed by the Deployment.  
- Label selectors in Deployment: The Deployment’s selector matches Pod labels, linking the controller to the Pods it owns, and misconfiguring this can break updates or scaling.  
- Service spec basics: A Service spec includes a type, a selector that picks up target Pods, and a `ports` list that maps Service ports to container ports.  
- Service types: `ClusterIP` exposes the Service inside the cluster, `NodePort` exposes it on node ports, and `LoadBalancer` integrates with cloud load balancers for external access.  
- Declarative configuration: Changing the YAML and re‑applying it triggers rolling changes through Kubernetes, instead of manually editing or recreating pods.  

---

## 7. Deploying a Multi-Component Application

- Multi‑component app structure: A real app is split into front‑end, back‑end, and database, each with its own Deployment and Service, promoting separation of concerns and independent scaling.  
- Internal service discovery: Front‑end and back‑end communicate via Service DNS names rather than IPs, allowing each layer to scale without configuration changes.  
- Configurations via ConfigMaps/Secrets: Application config (e.g., DB host, feature flags) is stored in ConfigMaps and Secrets and injected as env vars into Pods for each component.  
- Persistence layer: Databases or other stateful components use PersistentVolumeClaims so that data survives Pod restarts and upgrades, separating storage from compute.  
- Ingress for external routing: An Ingress resource fronts multiple Services, routing different HTTP paths or hostnames to specific back‑end Services under one public endpoint.  
- Namespaces for isolation: The demo often uses a dedicated namespace for the app, keeping its resources logically separated from system components and other workloads.  

---

## 8. Namespaces

- Namespaces as logical partitions: Namespaces allow you to group resources within one cluster, avoiding naming collisions and logically separating teams or environments.  
- Default and system namespaces: `default` is where resources go when no namespace is specified, while `kube-system` hosts core cluster components and should generally not be used for application workloads.  
- Access control with namespaces: RBAC policies can restrict users or service accounts to specific namespaces, providing a simple multi‑tenant isolation mechanism.  
- Resource quotas and limits: Namespaces can have quotas to cap CPU, memory, and object counts, preventing one team or app from consuming all cluster resources.  
- Environment separation: Dev, test, and prod can share the same physical cluster but run in different namespaces, each with its own configs, policies, and quotas.  
- Context and `-n` flag: You can switch default namespace via kubectl config or target a specific namespace per command with `-n`, making multi‑namespace workflows smoother.  

---

## 9. Ingress

- Ingress as HTTP gateway: Ingress is a Kubernetes resource that defines HTTP/S routing rules from the outside world to internal Services.  
- Path and host‑based routing: Ingress rules can send `/api` to one Service and `/web` to another, or route based on hostname (e.g., `api.example.com` vs `shop.example.com`).  
- Ingress controller requirement: An Ingress object alone does nothing; you must deploy an Ingress controller (like NGINX Ingress Controller) that reads Ingress resources and configures actual routing.  
- TLS termination: Ingress can terminate SSL/TLS at the edge, using Secrets holding certificates, so back‑end Services receive plain HTTP traffic.  
- Consolidated external access: Instead of exposing each Service individually via NodePort or LoadBalancer, Ingress lets you route many apps through one external IP or URL.  
- Annotations for advanced features: Ingress annotations and CRDs provided by controllers enable advanced behaviors like path rewrites, rate limiting, and authentication.  

---

## 10. Helm & Charts

- Helm as package manager: Helm is a tool that packages Kubernetes manifests into charts, allowing you to install, upgrade, and share complex applications easily.  
- Chart composition: A Helm chart bundles templates, default values (`values.yaml`), and metadata, making Kubernetes configuration parametrizable and reusable.  
- Values for customization: Users override default values to adapt a chart to different environments (dev, staging, prod) without editing the templates themselves.  
- Release abstraction: Each Helm installation of a chart is a release, tracked by Helm so you can upgrade, rollback, and inspect what’s running.  
- Ecosystem of charts: Popular software (databases, monitoring stacks, ingress controllers) is distributed as Helm charts, significantly reducing manual YAML writing.  
- Templating and DRY: Helm templates generate Kubernetes YAML dynamically based on values, reducing duplication and making large configurations easier to manage.  

---

## 11. Pod Networking (Container Communication)

- Shared network namespace in Pod: All containers in a Pod share the same IP and port space, so they can talk over `localhost` without any explicit Service or network configuration.  
- Sidecar pattern: Additional containers (sidecars) in the same Pod provide auxiliary functions (logging, proxying, configuration) for the main app container.  
- Inter‑container communication: Inside a Pod, containers can communicate via `localhost:<port>`, allowing them to collaborate tightly as if they were processes in the same machine.  
- Resource sharing trade‑off: Containers share CPU and memory limits defined at Pod level (or per container), making Pod design important for correct resource isolation.  
- Pod IP lifecycle: The Pod receives an IP from the cluster network, but that IP can change when the Pod is rescheduled, which is why external communication usually goes through Services.  

---

## 12. Persistent Storage (PV, PVC, StorageClass)

- Need for persistent storage: Stateless Pods can be recreated freely, but databases and other stateful apps require storage that outlives Pod lifecycles.  
- PersistentVolume (PV) concept: A PV is an abstraction over concrete storage (e.g., NFS share, cloud disk), representing a piece of storage available to the cluster.  
- PersistentVolumeClaim (PVC) concept: A PVC is a request for storage with a particular size and access mode, which Kubernetes binds to a suitable PV.  
- Decoupling storage and apps: Apps reference PVCs instead of storage details, so the same manifests work across environments with different underlying storage technologies.  
- StorageClass for dynamic provisioning: StorageClasses define how new PVs are provisioned on demand, enabling PVCs to automatically get backing storage without pre‑created PVs.  
- Mounting PVCs into Pods: Pods mount PVCs as volumes in specific container paths, giving containers a durable place to read/write data.  

---

## 13. Docker vs Swarm vs Kubernetes

- Docker role: Docker provides container building and runtime tools for individual hosts and forms the basis of many orchestrated environments.  
- Docker Swarm basics: Swarm is Docker’s simple orchestration layer, providing service definitions, scaling, and basic load balancing but with a smaller feature set.  
- Kubernetes as full orchestrator: Kubernetes is a more feature‑rich, extensible system offering advanced scheduling, self‑healing, autoscaling, and a large ecosystem.  
- Learning curve trade‑off: Swarm is easier to learn and set up but limited for complex production needs, while Kubernetes is more complex upfront but scales better to large, heterogeneous workloads.  
- Ecosystem and adoption: Kubernetes has become the de facto standard supported by all major cloud providers, tools, and vendors, making it a safer long‑term bet.  

---

## 14. Mounting ConfigMaps & Secrets

- ConfigMap as file volume: ConfigMaps can be mounted into Pods so that each key becomes a file and its value becomes the file content, ideal for config files.  
- Secret as file volume: Secrets similarly can be mounted as files, typically for certificates, keys, or credentials that applications read from the filesystem.  
- Env vars vs volume mounts: Config data can be injected either as environment variables or as files; volumes are better for large structured configs and things like SSL certificates.  
- Updating mounted configs: Changing a ConfigMap or Secret updates the underlying data, but Pods may need to be restarted or reloaded depending on how your app reads configuration.  
- Fine‑grained path mapping: You can control which keys get mounted and at which paths, allowing multiple configs or secrets to coexist cleanly inside a Pod’s filesystem.  

---

## 15. Pulling Private Images (imagePullSecrets)

- Private registry authentication in K8s: Kubernetes uses a special type of Secret to store Docker registry credentials (username, password, server).  
- Creating docker‑registry Secret: `kubectl create secret docker-registry` commands encode registry credentials into a Secret object in the appropriate namespace.  
- Referencing imagePullSecrets: Pod or Deployment specs include `imagePullSecrets` to tell kubelet which Secret to use when pulling private images.  
- Namespaced scope: Image pull Secrets must exist in the same namespace as the Pods that use them, enforcing per‑namespace authentication boundaries.  
- Avoiding hardcoding credentials: Using Secrets keeps credentials out of Pod specs and images, aligning with security best practices for managing sensitive data.  

---

## 16. StatefulSets vs Deployments

- Stateless nature of Deployments: Deployments assume that all Pods are interchangeable, without stable identities or storage, which suits stateless microservices.  
- StatefulSet requirement: StatefulSets are designed for stateful applications (like databases, Kafka) that need stable network identities and persistent, Pod‑bound storage.  
- Stable Pod names: Pods in a StatefulSet get predictable names (`app-0`, `app-1`, etc.), which some distributed systems use for cluster membership or replication roles.  
- Per‑Pod PVCs: Each StatefulSet replica gets its own PersistentVolumeClaim, ensuring that data and identity stick together across restarts and rescheduling.  
- Ordered operations: StatefulSets create and terminate Pods in a defined order (e.g., `app-0` before `app-1`), which is important for systems that rely on leader/follower or ordered startup.  
- Use‑case guidance: Databases, message queues, and consensus systems often rely on StatefulSets, while most stateless services remain on Deployments.  

---

## 17. Monitoring with Prometheus (Helm/Operator)

- Prometheus overview: Prometheus is a time‑series database and monitoring system that scrapes metrics from instrumented targets and supports alerting.  
- Prometheus Operator: The Prometheus Operator manages Prometheus instances using custom resources, simplifying configuration, upgrades, and service discovery in Kubernetes.  
- Helm install of monitoring stack: Using a Helm chart for the Prometheus Operator deploys Prometheus, Alertmanager, and related CRDs in a few commands instead of large manual YAML manifests.  
- Metrics scraping from K8s: The setup uses Kubernetes service discovery to automatically scrape metrics from nodes, Pods, and Services annotated for monitoring.  
- Grafana integration: Often Grafana is installed alongside Prometheus via Helm, providing dashboards to visualize cluster, application, and custom metrics.  
- Alerting pathways: Alertmanager, configured via the Operator, routes alerts to channels like email, Slack, or PagerDuty when metrics cross defined thresholds.  

---

## 18. Operators & CRDs

- Operator pattern idea: An Operator codifies operational knowledge about a specific app into a controller that manages that app via custom Kubernetes resources.  
- CustomResourceDefinitions (CRDs): CRDs extend the Kubernetes API with new resource types (e.g., `MyDatabase`) that the Operator understands and manages.  
- Custom controller logic: The Operator’s controller watches for CRD changes and performs complex tasks (backups, version upgrades, scaling rules) beyond what built‑in controllers do.  
- Domain‑specific automation: Operators embed human runbooks and best practices into software, reducing manual, error‑prone operational steps for mission‑critical systems.  
- User experience: Instead of scripting low‑level changes, users create or edit high‑level custom resources, and the Operator reconciles the system into the desired state.  

---

## 19. Service Types (ClusterIP, NodePort, LoadBalancer, Headless)

- ClusterIP Service: Default type that exposes a Service only inside the cluster via a virtual IP, used for internal communication between Pods.  
- NodePort Service: Allocates a fixed port on every node and forwards traffic from those ports to the Service, providing simple external access without cloud integration.  
- LoadBalancer Service: In cloud environments, automatically provisions an external load balancer and routes incoming traffic to the Service for production‑grade exposure.  
- Headless Service: A Service with `clusterIP: None` that returns Pod IPs directly via DNS, often used for StatefulSets and systems needing direct Pod‑to‑Pod awareness.  
- Label‑based selection: All Service types rely on label selectors to pick which Pods receive traffic, enabling easy blue‑green or canary patterns by switching labels.  
- DNS naming: Services get DNS names like `service-name.namespace.svc.cluster.local`, making it straightforward for Pods to connect without hard‑coded IPs.  
