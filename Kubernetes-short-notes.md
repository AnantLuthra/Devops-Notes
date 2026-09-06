# Kubernetes – Short Notes

### Main components of Kubernetes:
- Node - A machine on which pods run.
- Pod - A group of one or more containers, with shared storage/network, and a specification for how to run the containers.
- Control Plane - This is the collection of processes that manage the state of the cluster, including the API server, scheduler, and controller manager.
- Service - Through this we get a stable IP address and DNS name for a set of pods, allowing them to be accessed by other pods or external clients.
- Ingress - This is a collection of rules that allow inbound connections to reach the cluster services. We get a single IP address and DNS name for multiple services, and can also configure SSL termination and load balancing.
- Volume - This is a physical hard drive attachment to a node or a remote storage system that can be used by pods to store data persistently.
- ConfigMap - A Kubernetes object that lets you store configuration for your applications separately from the application code.
- Secret - A Kubernetes object that lets you store sensitive information, such as passwords, OAuth tokens, and SSH keys. (But it is not encrypted by default, so you should use additional measures to secure it.)

### Different ways of deployment in Kubernetes:
- Deployment - In this we configure desired number of replicas we want to run in a cluster. Kubernetes will ensure that the desired number of pods are running at all times, and will automatically replace any failed pods.
- StatefulSet - This deployment is for managing stateful applications, such as databases, that require stable network identities and persistent storage.
- DaemonSet - This makes sure that exactly one copy of a pod is running on each node in the cluster. It is useful for running background tasks, such as log collection or monitoring agents.


### 3 Services which must be present on worker node:
- Kubelet - This is an agent that runs on each node in the cluster and is responsible for managing the pods running on that node. It communicates with the Kubernetes API server to receive instructions and report back on the status of the pods.
- Kube-proxy - This is a network proxy that runs on each node in the cluster and is responsible for routing traffic to the appropriate pods based on the service definitions. It also handles load balancing and service discovery. First request comes to `service` and then kube-proxy routes it to the appropriate pod.
- Container runtime - This is the software that runs and manages the containers on each node. Kubernetes supports several container runtimes, including Docker(this also use containerd underneath), containerd, and CRI-O. The container runtime is responsible for starting and stopping containers, managing their lifecycle, and providing isolation between containers.


### Processes running on Control Plane:

1. API Server - This is the front-end for the Kubernetes control plane. It exposes the Kubernetes API and serves as the entry point for all administrative tasks. The API server validates and processes requests from users, controllers, and other components, and updates the cluster state accordingly.
2. Scheduler - This is responsible for assigning pods to nodes in the cluster based on resource availability, constraints, and policies. The scheduler watches for new pods that need to be scheduled and selects the best node for each pod based on factors such as CPU and memory usage, node affinity, and taints/tolerations. (This only decides which node to assign the pod to, in real kublet does all the work of creating the pod on that node.)
3. Controller Manager - This is a collection of controllers that manage the state of the cluster. It detects state changes, such as pods crashing. Each controller is responsible for a specific aspect of the cluster, such as managing replicas, ensuring that nodes are healthy, and handling events such as pod creation and deletion. The controller manager watches the cluster state and takes action to ensure that the desired state is maintained.
4. etcd - This is a key-value store that is used by the Kubernetes control plane to store all the cluster data. It is a critical component of the control plane and must be highly available and reliable.


### Minikube & kubectl - local kubernetes setup:

- Minikube - It is a one node cluster in which Control Plane and Worker Node processes run on the same machine. It is used for local development and testing of Kubernetes applications. It is docker container runtime preinstalled.

- Kubectl - It is a command line tool that allows you to interact with the Kubernetes cluster (As we know that there are 3 ways in which we can intract with API Server of Control Plane, i.e. - CLI, UI, API. So kubectl is the CLI way of interacting with API Server). It allows you to deploy and manage applications, inspect cluster resources, and view logs. It communicates with the API server using RESTful APIs and supports a wide range of commands for managing Kubernetes resources. This works for cloud or hybrid clusters as well.


### Kubectl commands:

The Flow: We manage Deployements -> which creates replica sets -> which creates pods -> pods manages containers. So everything is abstract we only need to manage deployments and kubernetes will take care of the rest. So we can use below commands to manage deployments and pods.

- `kubectl get pods` - This command lists all the pods in the current namespace. It shows the name, status, and other details of each pod.
- `kubectl describe pod <pod-name>` - This command shows detailed information about a specific pod, including its containers, volumes, events, and resource usage.
- `kubectl logs <pod-name>` - This command shows the logs of a specific pod. It can be useful for debugging issues with the pod or its containers.
- `kubectl exec -it <pod-name> -- <command>` - This command allows you to execute a command inside a specific pod. It can be useful for troubleshooting or running administrative tasks inside the pod. Example - `kubectl exec -it <pod-name> -- /bin/bash` - This command opens a shell inside the pod, allowing you to interact with the pod's file system and run commands as if you were logged in to the pod directly.
- `kubectl apply -f <file-name>` - This command applies a configuration file to the cluster. It can be used to create or update resources such as pods, services, and deployments. The configuration file can be in YAML or JSON format.
- `kubectl delete -f <file-name>` - This command deletes resources defined in a configuration file from the cluster. It can be used to remove pods, services, deployments, and other resources. The configuration file can be in YAML or JSON format.
- `kubectl get services` - This command lists all the services in the current namespace. It shows the name, type, cluster IP, external IP, and other details of each service.
- `kubectl get replicasets` - This command lists all the ReplicaSets in the current namespace. It shows the name, desired replicas, current replicas, and other details of each ReplicaSet.


### Deployement and Service YAML file:

It has 3 parts - Metadata, Spec, and Status. We only need to define Metadata and Spec in our YAML file. Status is automatically generated by etcd store in Kubernetes. And yaml is very strict about indentation.

Below is yaml file for deployment of nginx pod with 2 replicas and service to route the request to the pod.

#### Deployment YAML file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 8080
```

##### Service YAML file:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

- So every thing has a label, and pod's label has to match with deployment's selector label.
- Deployment's label is used to identify the deployment, and it can be used to filter and group resources in the cluster.
- Service's selector label has to match with pod's label. So that service can route the request to the pod.
- Service's targetPort has to match with pod's containerPort. So that service can route the request to the pod's container.
- We can use `kubectl get pods --show-labels` to see the labels of pods. 
- `kubectl get pods -o wide` or `kubectl describe pods <pod-name>` to see the node on which pod is running.
- `kubectl delete -f nginx-deployment.yaml` - This command deletes the deployment and all the pods created by it.
- `kubectl delete -f nginx-service.yaml` - This command deletes the service and all the pods created by it.

#### Yaml file got from etcd store of Kubernetes for the above deployment and service is as below:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.25","name":"nginx","ports":[{"containerPort":8080}]}]}}}}
  creationTimestamp: "2026-09-06T19:03:09Z"
  generation: 1
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "13872"
  uid: a21e4087-679f-4c92-8c39-1fdeb7806205
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.25
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2026-09-06T19:03:15Z"
    lastUpdateTime: "2026-09-06T19:03:15Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-09-06T19:03:09Z"
    lastUpdateTime: "2026-09-06T19:03:15Z"
    message: ReplicaSet "nginx-deployment-7577994b67" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 2
  replicas: 2
  terminatingReplicas: 0
  updatedReplicas: 2
```