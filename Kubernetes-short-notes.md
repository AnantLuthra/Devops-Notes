# Kubernetes – Short Notes

Index
- 

## Main components of Kubernetes:
- Node - A machine on which pods run.
- Pod - A group of one or more containers, with shared storage/network, and a specification for how to run the containers.
- Control Plane - This is the collection of processes that manage the state of the cluster, including the API server, scheduler, and controller manager.
- Service - Through this we get a stable IP address and DNS name for a set of pods, allowing them to be accessed by other pods or external clients.
- Ingress - This is a collection of rules that allow inbound connections to reach the cluster services. We get a single IP address and DNS name for multiple services, and can also configure SSL termination and load balancing.
- Volume - This is a physical hard drive attachment to a node or a remote storage system that can be used by pods to store data persistently.
- ConfigMap - A Kubernetes object that lets you store configuration for your applications separately from the application code.
- Secret - A Kubernetes object that lets you store sensitive information, such as passwords, OAuth tokens, and SSH keys. (But it is not encrypted by default, so you should use additional measures to secure it.)

## Different ways of deployment in Kubernetes:
- Deployment - In this we configure desired number of replicas we want to run in a cluster. Kubernetes will ensure that the desired number of pods are running at all times, and will automatically replace any failed pods.
- StatefulSet - This deployment is for managing stateful applications, such as databases, that require stable network identities and persistent storage.
- DaemonSet - This makes sure that exactly one copy of a pod is running on each node in the cluster. It is useful for running background tasks, such as log collection or monitoring agents.


## 3 Services which must be present on worker node:
- Kubelet - This is an agent that runs on each node in the cluster and is responsible for managing the pods running on that node. It communicates with the Kubernetes API server to receive instructions and report back on the status of the pods.
- Kube-proxy - This is a network proxy that runs on each node in the cluster and is responsible for routing traffic to the appropriate pods based on the service definitions. It also handles load balancing and service discovery. First request comes to `service` and then kube-proxy routes it to the appropriate pod.
- Container runtime - This is the software that runs and manages the containers on each node. Kubernetes supports several container runtimes, including Docker(this also use containerd underneath), containerd, and CRI-O. The container runtime is responsible for starting and stopping containers, managing their lifecycle, and providing isolation between containers.


## Processes running on Control Plane:

1. API Server - This is the front-end for the Kubernetes control plane. It exposes the Kubernetes API and serves as the entry point for all administrative tasks. The API server validates and processes requests from users, controllers, and other components, and updates the cluster state accordingly.
2. Scheduler - This is responsible for assigning pods to nodes in the cluster based on resource availability, constraints, and policies. The scheduler watches for new pods that need to be scheduled and selects the best node for each pod based on factors such as CPU and memory usage, node affinity, and taints/tolerations. (This only decides which node to assign the pod to, in real kublet does all the work of creating the pod on that node.)
3. Controller Manager - This is a collection of controllers that manage the state of the cluster. It detects state changes, such as pods crashing. Each controller is responsible for a specific aspect of the cluster, such as managing replicas, ensuring that nodes are healthy, and handling events such as pod creation and deletion. The controller manager watches the cluster state and takes action to ensure that the desired state is maintained.
4. etcd - This is a key-value store that is used by the Kubernetes control plane to store all the cluster data. It is a critical component of the control plane and must be highly available and reliable.


## Minikube & kubectl - local kubernetes setup:

- Minikube - It is a one node cluster in which Control Plane and Worker Node processes run on the same machine. It is used for local development and testing of Kubernetes applications. It is docker container runtime preinstalled.

- Kubectl - It is a command line tool that allows you to interact with the Kubernetes cluster (As we know that there are 3 ways in which we can intract with API Server of Control Plane, i.e. - CLI, UI, API. So kubectl is the CLI way of interacting with API Server). It allows you to deploy and manage applications, inspect cluster resources, and view logs. It communicates with the API server using RESTful APIs and supports a wide range of commands for managing Kubernetes resources. This works for cloud or hybrid clusters as well.


## Kubectl commands:

The Flow: We manage Deployements -> which creates replica sets -> which creates pods -> pods manages containers. So everything is abstract we only need to manage deployments and kubernetes will take care of the rest. So we can use below commands to manage deployments and pods.

- `kubectl get pods` - This command lists all the pods in the current namespace. It shows the name, status, and other details of each pod.
- `kubectl describe pod <pod-name>` - This command shows detailed information about a specific pod, including its containers, volumes, events, and resource usage.
- `kubectl logs <pod-name>` - This command shows the logs of a specific pod. It can be useful for debugging issues with the pod or its containers.
- `kubectl exec -it <pod-name> -- <command>` - This command allows you to execute a command inside a specific pod. It can be useful for troubleshooting or running administrative tasks inside the pod. Example - `kubectl exec -it <pod-name> -- /bin/bash` - This command opens a shell inside the pod, allowing you to interact with the pod's file system and run commands as if you were logged in to the pod directly.
- `kubectl apply -f <file-name>` - This command applies a configuration file to the cluster. It can be used to create or update resources such as pods, services, and deployments. The configuration file can be in YAML or JSON format.
- `kubectl delete -f <file-name>` - This command deletes resources defined in a configuration file from the cluster. It can be used to remove pods, services, deployments, and other resources. The configuration file can be in YAML or JSON format.
- `kubectl get services` - This command lists all the services in the current namespace. It shows the name, type, cluster IP, external IP, and other details of each service.
- `kubectl get replicasets` - This command lists all the ReplicaSets in the current namespace. It shows the name, desired replicas, current replicas, and other details of each ReplicaSet.


## Deployment and Service YAML file:

It has 3 parts - Metadata, Spec, and Status. We only need to define Metadata and Spec in our YAML file. Status is automatically generated by etcd store in Kubernetes. And yaml is very strict about indentation.

Below is yaml file for deployment of nginx pod with 2 replicas and service to route the request to the pod.

### Deployment YAML file:
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

#### Service YAML file:
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

### Yaml file got from etcd store of Kubernetes for the above deployment and service is as below:
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

## Namespaces:

There are two namespaces:
1. **Namespace in Linux** - It is a feature of the Linux kernel that allows you to create isolated environments for processes. Each namespace has its own set of resources, such as process IDs, network interfaces, and file systems. This allows you to run multiple instances of the same application on the same machine without them interfering with each other. Example - A docker container gets its own namespace for process IDs, network interfaces, and file systems, so that it can run independently of other containers on the same machine.
   - Why it is needed - It is needed to provide isolation between different applications running on the same machine. Without namespaces, all processes would share the same resources, which could lead to conflicts and security issues. By using namespaces, you can ensure that each application has its own isolated environment, which improves security and stability.
2. **Namespace in Kubernetes** - It is a way to divide cluster resources between multiple users.
## Why we use namespaces

 1. **Hard to manage:** Everything in one default namespace will be messy and hard to manage. Instead of that we can one namespace for `database`, one for `monitoring`, `elastic stack`, `nginx-ingress` and so on. This will make it easier to manage and organize resources in the cluster.
 2. **Name Conflicts in teams:** Team A had a deployement with abc.yaml name, later Team B did kubectl apply -f abc.yaml which will cause a conflict.
 3. **Resources Sharing:** 
    1. We can use same resources i.e. - `nginx-ingress controller` , `elastic stack` for both staging and production namespaces. This will save resources and cost.
    2. Blue green deployment - We can use two namespaces for blue and green deployments. This will allow us to deploy new versions of our application without affecting the existing version. We can switch between the two versions by changing the service selector to point to the new version.
 4. **Resources Quotas:** We can set resource quotas for each namespace, which limits the amount of CPU, memory, and storage that can be used by the resources in that namespace. This helps to prevent one team from consuming all the resources in the cluster and ensures that resources are allocated fairly among different teams.

## Characteristics of Namespaces in Kubernetes:
- We cannot use most of the resources from another namespace. For example, we can't use a configmap(that is using db (which itself is in a different namespace)) of a namespace from another namespace.
- But we can use database service from a configmap of a namespace from another namespace. For example, we can use a configmap of a namespace which is using database service of another namespace.
- Components of Kubernetes that are not namespaced include nodes, persistent volumes, storage classes, and namespaces themselves. These resources are global to the cluster and are not associated with any particular namespace.
  - Command to see all the resources which are not namespaced - `kubectl api-resources --namespaced=false`
  - Command to put a service in a namespace - `kubectl create service <service-name> --namespace=<namespace-name>`. This will create a service in the specified namespace.
  - We can also put namespace in the yaml file of the service. Example - 
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
    namespace: my-namespace
  ```
  - We can also use `kubectl get all --namespace=<namespace-name>` to see all the resources in a specific namespace. This will show us all the pods, services, deployments, and other resources that are associated with that namespace.
  - **Important**: Kubectl by default looks for resources in the `default` namespace. To change this we can use `kubectl config set-context --current --namespace=<namespace-name>`. This will set the current context to the specified namespace, and all subsequent kubectl commands will be executed in that namespace. We can also use `kubectl config view --minify | grep namespace:` to see the current namespace that is set in the context.
  - Command to create a namespace - `kubectl create namespace <namespace-name>`. This will create a new namespace in the cluster. We can also use `kubectl get namespaces` to see all the namespaces in the cluster. 
  - Install `kubectx` which will give access to `kubens` command which will allow us to switch between namespaces easily. Example - `kubens <namespace-name>` will switch to the specified namespace.

## Different types of Namespaces in Kubernetes:
1. **Default Namespaces** - These are the namespaces that are created by default when you create a Kubernetes cluster. They include:
- `default` - This is the default namespace for resources that are not assigned to any other namespace.
- `kube-system` - This namespace is used for resources that are managed by the Kubernetes system, such as the API server, scheduler, and controller manager.
- `kube-public` - This namespace is used for resources that are publicly accessible, such as the Kubernetes dashboard.
- `kube-node-lease` - This namespace is used for resources that are related to node leases, which are used to track the availability of nodes in the cluster.
2. **User-defined namespaces** - These are the namespaces that you can create to organize your resources in a way that makes sense for your application. You can create as many user-defined namespaces as you need, and you can assign resources to them using labels and selectors.

## IP addresses for nodes and pods
- Each node gets a range of IP addresses for eg. 
  - Node 1 - 10.1.1.x
  - Node 2 - 10.1.2.x
- And each pod gets an IP from the range in which node it is running.
  - For example the pod running on node 1 can have `10.1.1.5` as ip address.

## Services


### ClusterIP Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  ports:
  - name: application
    protocol: TCP
    port: 8081
    targetPort: 8081
  - name: logs-exporter
    protocol: TCP
    port: 8082
    targetPort: 8082
```

- Above is an example of clusterIP - which is a multiport service.
  - **Multiport Service**: If you have 2 containers in a pod, and their ports are 8081 and 8082, so then we need to open 2 ports in the service so that it can forward it to both of them.
    - `port` - This is the port on which service listens and `targetPort` - is the port on which service will forward the request to the pod - and this is the port on which container is also listening.

- And when `service` is called from `ingress` it is mentioned in the yaml file of ingress as below:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mongo-express-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: mongo-express.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mongo-express-service
            port:
              number: 8081
```

Above 8081 is the port on which service is listening, and we've also mentioned the name of service accordingly.

## Headless Service:
We use this type when we wanna access a specific pod directly - instead of service load balancing to any pod.
For example in a stateful application - its database replicas would be different at a time - so lets say now we wanna create another replica, then we need to access the last updated replica directly, for that we create a headless service through which we'll do a `DNS lookup` to get the IP of the pod we wanna access directly. 

Example of headless service yaml file is as below:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  clusterIP: None
  selector:
    app: mongo-express
  ports:
    protocol: TCP
    port: 8081
    targetPort: 8081
```
In above code - `clusterIP: None` will not assign an IP to the service, and it will not load balance the request to any pod. Instead it will return the IP of the pod which is running the application.

There can also be a case where both clusterIP and headless services are used - one for load balancing and another for direct access to a specific pod. 

## NodePort Service:
- In this type of service - we open a static port on each worker node in our k8s cluster - and that port is accessable from internet. So when we call that port from internet - it will be routed to the service and then service will route it to the pod. 
- This is insecure because we're exposing our application to the external traffic directly, and anyone can access it. So we use this type of service only for testing purposes.
- Example of nodeport service yaml file is as below:
```yaml
apiVersion: v1
kind: Service 
metadata:
  name: mongo-express-service
spec:
  type: NodePort
  selector:
    app: mongo-express
  ports:
    protocol: TCP
    port: 8081
    targetPort: 8081
    nodePort: 30000
```
- In above code - `nodePort: 30000` is the port which is opened on each worker node. It has a fixed range of 30000-32767.
- We need to put `type: NodePort` in the yaml file to make it a nodeport service. Otherwise it will be a clusterIP service by default.
- **Important**: Creating this NodePort service will automatically create a clusterIP service as well, which will be used by the nodeport service to route the request to the pod.
- Therefore with a NodePort Service, the application can be reached in two ways:
  1. **Inside the cluster:**
     - `ClusterIP:port` or `service-name:port`
  2. **From outside the cluster:**
     - `NodeIP:nodePort`



## LoadBalancer Service:

- This is same to NodePort service - its just that the port opened by Loadbalancer is only accessible from the cloud provider's load balancer. So we can use this type of service in production environment. Rest of the things are same as NodePort service.
- And creating this LoadBalancer will automatically create a NodePort, ClusterIP as well.
- In this we pass `type: LoadBalancer` in the yaml file to make it a loadbalancer service. Otherwise it will be a clusterIP service by default.

## Ingress:
We use ingress to make an appraction from `service` and have a domain name with SSL certificate. So when we call the domain name from internet instead of a ip with a port.

- **Difference between external and internal service** - Is that we don't have a 3rd port opened in internal one.

Example of Ingress yaml file is as below:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mongo-express-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mongo-express
            port:
              number: 8081
```
Things to note about above code.
- `host: myapp.com` - myapp.com has to be a valid domain name which is registered and has a valid SSL certificate. And it should be pointing to the IP of the entrypoint node of the cluster. Entry point can also be a server outside the kubernetes cluster.
- We need Implementation of Ingress - which is Ingress Controller.
- The request can be coming from an ingress controller from different places..
  1. If our k8s cluster is running on a cloud provider - then the request can be coming from the cloud provider's load balancer.
  2. If our k8s cluster is running on a bare mental machine - then the request can be coming from a software or a hardware solution which might be inside of the cluster or outside as a seperate server. For example - Nginx, HAProxy, Traefik, Istio, Envoy, etc. And example of seperate server would be a proxy server.
      - Flow incase of a proxy server - Request comes to opened ports of proxy server -> passed to ingress controller -> then it will smartly forward it to required service -> then service will forward it to the pod.


## Configuring Ingress in Minikube:
- If we have setup in Minikube we can simple run `minikube addons enable ingress` to enable ingress controller in minikube. This will install Nginx ingress controller in the cluster. With this following things happen:
  - Automatically starts the K8s Nginx implementation of Ingress Controller
  - Minikube uses minikube tunnel to enable ingress access
  - Ingress Controller configured with just one command
Ingress yaml example
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
spec:
  rules:
  - host: dashboard.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service: 
              name: kubernetes-dashboard
              port: 
                number: 80
```

- Above is an ingress for command `minikube dashboard` - it installs and runs kubernetes dashboard.
and for making it available on my browser as `dashboard.com` we need to add an entry in our hosts file as below:
```
127.0.0.1 dashboard.com
```
- And then we need to make a tunnel to the ingress controller by running `minikube tunnel` command in a separate terminal. This will create a tunnel between our local machine and the ingress controller which is inside the docker container in which our minikube is running. And then we can access the dashboard by going to `dashboard.com` in our browser.
- If case of having this minikube setup in WSL - you need to put `127.0.0.1` in hosts file of WSL and windws both. Because the request will be coming from windows to WSL and then to minikube docker container. So we need to make sure that the request is routed correctly.

**So the flow will be.**
1. **Domain Resolution (/etc/hosts)**
- When you type `http://dashboard.com` in your browser, your system checks the local hosts file before querying public DNS. Because you mapped 127.0.0.1 dashboard.com, the browser bypassed the internet and sent the request to your local machine `(localhost:80)`.

2. **Traffic Bridging (minikube tunnel)**
- Your Kubernetes cluster runs inside a Docker container with an isolated network `(192.168.49.2)`. Your Windows/WSL host cannot reach that internal subnet directly.
- minikube tunnel acts as a networking bridge: it binds ports `80` and `443` on your host machine `(127.0.0.1)` and forwards any incoming packets straight to Minikube's internal network gateway.

3. **Routing via Ingress Controller**
- Inside Minikube, the Nginx Ingress Controller listens for traffic entering on port 80:
- It inspects the incoming HTTP header: `Host: dashboard.com`.
- It matches this against your Ingress rule: `host: dashboard.com` and `path: /`.
- It routes the request to the backend Service `kubernetes-dashboard` on port 80.

4. **Service to Pod**
- The `kubernetes-dashboard` Service receives the request and forwards it to the active Dashboard Pod running inside the `kubernetes-dashboard` namespace, which returns the web UI back to your browser.

### Multiple Paths from a same host in Ingress:

Example of ingress yaml file is as below:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /dashboard
        pathType: Prefix
        backend:
          service:
            name: dashboard-service
            port:
              number: 8081
      - path: /statistics
        pathType: Prefix
        backend:
          service:
            name: statistics-service
            port:
              number: 8082
```
### Multiple Domains or Subdomains from a same ingress controller:
Example of ingress yaml file is as below:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-domain-ingress
spec:
  rules:
  - host: dashboard.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: dashboard-service
            port:
              number: 8081
  - host: statistics.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: statistics-service
            port:
              number: 8082
```

### Adding SSL certificate to Ingress:
- We can add SSL certificate to ingress by adding `tls` section in the ingress yaml file
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ssl-ingress
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 8080
```

Yaml code of secret which contains the SSL certificate and private key is as below:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-tls
type: kubernetes.io/tls
data:
  tls.crt: base64 encoded certificate
  tls.key: base64 encoded private key
```
One thing to keep in mind is that the secret should be created in the same namespace as the ingress resource. Otherwise, the ingress controller will not be able to find the secret and will fail to configure SSL for the specified host.


## Persistent Storage in Kubernetes:

Our storage should be such which follows these requirements:
1) Storage that doesn't depend on the pod lifecycle.
2) Storage must be available on all nodes.
3) Storage needs to survive even if cluster crashes.

**Things about Persistent Storage in Kubernetes:**
- Kubernetes doesn't manage the storage itself, it just provides an abstraction layer for storage. So we can use any storage solution that meets the above requirements.
- We create a PersistentVolume with YAML file just like other resources in Kubernetes. 
- We can store data in an actual hard drive attached to nodes or a nfs volume, or a cloud storage solution like AWS EBS, GCP Persistent Disk, Azure Disk, etc. So pods can use different storage solutions without changing the pod configuration.
- Storage aren't namespaced resources, so we can use a PersistentVolume from any namespace. 


### Local VS Remote Persistent Storage:

Local persistent storage failes these 2 requirements:
- Being tied to 1 specific node
- Not surviving cluster crashes

There we should almost everytime use remote storage.

Example of NFS PersistentVolume yaml file is as below:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-name
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsers=4.0
  nfs:
    path: /dir/path/on/nfs/server
    server: nfs-server-ip-address
```

Example of Google Cloud Persistent Disk PersistentVolume yaml file is as below:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
  labels:
    topology.kubernetes.io/zone: us-central1-a__us-central1-b
spec:
  capacity:
    storage: 400Gi
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4
```

- So Generally system administrators create the actual persistant storage and the whole cluster based on the needs of developers, these are SREs or devops engineers.
- And second comes the developers & devops engineers who create PersistentVolumeClaim to use the storage in their pods. 
- Developers write the YAML file for the PersistentVolumeClaim and the pod which will use it. And then they apply it to the cluster. The cluster will then bind the PersistentVolumeClaim to a PersistentVolume that matches the requirements of the claim. And then the pod will be able to use the storage.

Example of PersistentVolumeClaim yaml file is as below:
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-name
spec:
  storageClassName: manual
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Example of how we would use this claim in our pod yaml file is as below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: myimage
    volumeMounts:
    - mountPath: "/data"
      name: mypvc
  volumes:
  - name: mypvc
    persistentVolumeClaim:
      claimName: pvc-name
```

**Important Note**: PVs aren't namespaced resources, so we can use a PersistentVolume from any namespace. But PVCs are namespaced resources, so we can only use a PersistentVolumeClaim from the same namespace as the pod.

- **Level of Abstraction**: From Bottom to Top - PersistentVolume -> PersistentVolumeClaim -> Pod -> Container. So we can say that PersistentVolume is the actual storage, PersistentVolumeClaim is the request for storage, and Pod is the consumer of storage.
- We can also mount a configmap or a secret as a volume in a pod. This allows us to store configuration data or sensitive information in a separate resource and mount it into the pod at runtime. This is useful for separating configuration from code and for managing secrets securely. Example below.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: busybox-containers
      image: busybox
      volumeMounts:
        - name: config-diir
          mountPath: /etc/config
  volumes:
    - name: config-dir
      configMap:
        Name: bb-configmap
```

## Storage Classes in Kubernetes:
- Storage class provisions Persistent Volumes dynamically, when PersistentVolumeClaim Claims it.

Yaml for creating a storage class is as below:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-class-name
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGb: "10"
  fsType: ext4
```

So when we create a PersistentVolumeClaim with the storage class name, it will automatically create a PersistentVolume with the specified parameters. And then the pod can use that PersistentVolumeClaim to access the storage.

## Mounting ConfigMap and Secret as Volumes in Pods:

Example of mounting a ConfigMap, Secret as a volume in a pod is as below:
```yaml
spec:
        containers:
          - name: mosquitto
            image: eclipse-mosquitto:2.0
            ports:
              - containerPort: 1883
            volumeMounts:
              - name: mosquitto-config
                mountPath: /mosquitto/config
              - name: mosquitto-secret
                mountPath: /mosquitto/secret
                readOnly: true
              
        volumes:
          - name: mosquitto-config
            configMap: 
              name: mosquitto-config-file
          - name: mosquitto-secret
            secret: 
              secretName: mosquitto-secret-file
```

## Different Between StatefulSet and Deployment

1. In deployment we have stateless applications - which application pods can be scaled up and down easily, and they don't have any persistent state. But in statefulset we have stateful applications - which application pods have a persistent state, and they need to be scaled up and down carefully.
2. Even database pod in stateless application can't be scaled up easily because data needs to be consistent across the pods.
3. Also in deployment the app can have request in random order through Service, but in a database pod where replicas are there - request has to go through specific pod because the data is not consistent across the pods. 
4. Stateful applications aren't suitable for containerization, but stateless applications can easily be containerized.

**What Statefullset provides:**
1. Stable, unique network identifiers - Each pod in a StatefulSet has a sticky unique name(identifier) and a fixed individual DNS name, e.g., `mysql-0.svc2`, first is statefulsetname, then pod number, then service name. 
2. In statefullset the name of pods are predictable and ordered, so we can use them to access the pods directly. For example - if we have a statefulset with 3 replicas, the pods will be named as `pod-0`, `pod-1`, and `pod-2` unlike deployment where the pods are named randomly.
3. In statefull set pods are created in order, and they are terminated in reverse order. So if we have a statefulset with 3 replicas, the pods will be created in the order of `pod-0`, `pod-1`, and `pod-2`, and they will be terminated in the order of `pod-2`, `pod-1`, and `pod-0`. This is important for stateful applications because they need to be scaled up and down carefully to maintain data consistency.


**What happens in database applications with statefullsets:**
- As we have 1 main database, and others are replicas of that which are only used for read operations. So each replica has to maintain a sync with the main pod.
- We use PersistentVolumeClaims to store the data of each pod in a StatefulSet. Each pod gets its own PersistentVolumeClaim, which contains the replicated data, and its own state. Through this mechanism, even if the pod dies, it can be rescheduled, and to make it work we should use remote persistent volume because a pod can be rescheduled on a different node, and the data should be available on that node as well. 

## Managed Vs Unmanaged Kubernetes Cluster:

1. Create own cluster from scratch - In this we have to create our own cluster from scratch, and we have to manage the cluster ourselves. We have to install and configure the Kubernetes components, and we have to manage the cluster ourselves. This is a good option if we want to have full control over the cluster, and we have the expertise to manage it. But it is a lot of work, and it is not recommended for production environments.
2. Managed k8s cluster - In this we use a managed Kubernetes service from a cloud provider, such as AWS EKS, GCP GKE, Azure AKS, etc. The cloud provider manages the cluster for us, and we don't have to worry about the underlying infrastructure. This is a good option if we want to focus on our application development, and we don't want to worry about managing the cluster. But it is more expensive than creating our own cluster from scratch.

**Managed Example. - Linode Kubernetes Engine (LKE)**
- You only care about Worker Nodes
- Everything pre-installed
- Control Plane Nodes created and managed by Cloud Provider
- you only pay for the Worker Nodes
- Less effort and time

**Example.**
- Lets say we wanna run a mongodb pod in LKE, we will select number of worker nodes and their type cpu ram etc, and select region.
- Now we need to make persistent storage as kubernetes doesn't provide us - so we'll need to..
  - Create physical storage
  - Create persistent volume
  - Attach volumes to your database
- But instead we can use Linode Block Storage in which linode creates:
  - Persistent Volumes
  - with physical storage
- Once we have our Node app running, MongoDB pod running, storage configured, now we need services and ingress to make our application accessible from internet. For that we use Linode NodeBalancer.
- Linode's LoadBalancer comes in front of our enginx ingress controller, so that becomes the entrypoint for our application. 

**Things can be done with Linode's LoadBalancer:**
- Scaling up and down the application easily.
- Adding Session stickiness - in which if your application saves some data in a pod, so this stickiness will pass next request from that same user to that same pod only.
- Adding SSL certificate to the LoadBalancer - Done with the help of `cert-manager` plugin, and store the ssl or tls certificate in a k8s secret.
- We can also move our nodes closer to our users by changing the region of the nodes, and this will reduce the latency of our application.

**Important Notes**
- **Vender Lock-in** - If we someday decide to complete or part of our application to another cloud provider, we will have to do a lot of work to move our application and data to the new cloud provider.
- We can automate tasks using Terraform, Ansible. - to save time and work more efficiently.

## Helm
- **What is helm?** - Helm is a package manager for Kubernetes. It allows you to define, install, and upgrade even the most complex Kubernetes applications.
- **What are helm Charts?** - Helm charts are a collection of yaml files that are pushed by differnet people or Organizations to helm repositories - and those yaml files are for different tools and applications, just like we get docker images from docker hub. Example would be downloading helm charts for `nginx-ingress` controller, `cert-manager`, `prometheus`, `grafana`, etc. And we can use those charts to install those applications in our cluster.
- We can do `help search <chart-name>` to search for a chart in the helm repository, and then we can do `helm install <release-name> <chart-name>` to install that chart in our cluster. And we can also do `helm upgrade <release-name> <chart-name>` to upgrade that chart to a new version.
- **Templating Engine?** - If we've many microservices or any such case where we're writing many yaml files and most of there code is same, just verions or names are different, in that case we use help templating and put dynamic values which are replaced by placeholders.
Example of that template yaml file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Values.name }}
spec:
  containers:
    - name: {{ .Values.container.name }}
      image: {{ .Values.container.image }}
      port: {{ .Values.container.port }}
```

Now the files from which these values are coming is - `values.yaml` file which is as below:
```yaml
name: my-app
container:
  name: my-app-container
  image: my-app-image
  port: 9001
```

- **Another use of Helm is to deploy the same applications across different environments** - with the help of help charts, so we will create our own application chart that will have all the yaml files. Then we can use that same chart to deploy our application in different environments - like dev, staging, production, etc. And we can use different values.yaml files for each environment to customize the deployment.

Helm Chart Structure:
```
mychart/
  Chart.yaml          # Information about your chart
  values.yaml         # The default values for your templates
  charts/             # Charts that this chart depends on
  templates/          # The template files
```

- When we do `helm install <release-name> <chart-name>` - it will create a release of that chart in our cluster - it will take templates from templates folder and insert values from values.yaml.
- Ways to override values provided in values.yaml file:
  1. `--set` flag - We can use `--set` flag to override values provided in values.yaml file. Example - `helm install <release-name> <chart-name> --set name=my-app --set container.name=my-app-container --set container.image=my-app-image --set container.port=9001`
  2. `-f` flag - We can use `-f` flag to provide a custom values.yaml file. Example - `helm install <release-name> <chart-name> -f custom-values.yaml`

- **Release Management**
  - `helm install <release-name> <chart-name>` - Install a new release of a chart
  - `helm upgrade <release-name> <chart-name>` - Upgrade an existing release of a chart
  - `helm rollback <release-name> <revision>` - Rollback to a previous release of a chart


## Deploying Images in Kubernetes from private docker repository:

### Step 1
- Do docker login: Create config.json file for Secret - That will be created when you do `docker login` command. This file is created in `~/.docker/config.json` and it contains the authentication information for the private docker registry. We will use this file to create a secret in Kubernetes.

- So we'll create a Secret using this config.json file, either through a yaml file - or through a command if you don't wanna mention the actual content of config.json file in the yaml file. Example of command is as below:
```bash
kubectl create secret generic my-registry-key --from-file=.dockerconfigjson=/path/to/config.json --type=kubernetes.io/dockerconfigjson
```
The way we can make this secret using YAML file is as below:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-registry-key
data:
  .dockerconfigjson: aoaiwejfi2983fjakjw9ofij2o #base 64 encoded content of config.json file
type: kubernetes.io/dockerconfigjson
```

Or we can do docker login - and create secret both at the same time using this command below:
```bash
kubectl create secret docker-registry my-registry-key --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```
Example command for above:
```bash
kubectl create secret docker-registry my-registry-key --docker-server=https://index.docker.io/v1/ --docker-username=myusername --docker-password=mypassword
```

### Step 2
- Using that secret in our Deployment yaml file to pull the image from private docker registry. Example of that is as below:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-two
  labels:
    app: my-app-two
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app-two
  template:
    metadata:
      labels:
        app: my-app-two
    spec:
      imagePullSecrets:
      - name: my-registry-key-two
      containers:
      - name: my-app-two
        image: IMAGE_NAME_HERE
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```

**Important Note:**
- Secret in which we store the credentials and the deployment where we're accessing that secret should be in the same namespace. Otherwise, the deployment will not be able to access the secret.

## Operators in Kubernetes:

- Operators get into picture because incase of stateless applications it is easy for core k8s to manage self heal and do all those things, but incase of stateful applications we still get to need a human intervention to manage the stateful applications. 
- So Operators are another self healing loop or logic which does the following:
  - How to create the mysql cluster
  - How to run it
  - How to synchronize the data
  - How to update
- We can look for operators developed by community from `operatorhub` or `github` and use them.
- If we wanna create our own operator - we can use `operator-sdk` which is a tool that helps us to create operators easily. It provides us with a framework to create operators in golang, ansible, or helm. And it also provides us with a set of libraries and tools to help us to create operators easily.

## Kubernetes API Groups Reference Notes

In Kubernetes, the API is organized into modular categories called **API Groups**. 
Each resource belongs to an API group, which determines:
1. The REST API path used by `kubectl` and the API server (`/api/v1` vs `/apis/<group>/<version>`).
2. The `apiVersion` field at the top of your YAML manifests.
3. The `apiGroups` list inside RBAC resources (`Role`, `ClusterRole`).

---

### 1. The Core Group (`""`)

The Core group represents the earliest, fundamental building blocks of Kubernetes.

- **RBAC Syntax:** Represented as an empty string `[""]`.
- **API URL Path:** `/api/v1` (does not use `/apis/`).
- **YAML `apiVersion`:** Simply `v1`.

#### Common Resources:
- `pods`, `pods/log`, `pods/exec`, `pods/status`
- `services`, `endpoints`
- `namespaces`
- `configmaps`
- `secrets`
- `persistentvolumes` (pv), `persistentvolumeclaims` (pvc)
- `serviceaccounts`
- `nodes`

---

### 2. Named API Groups

As Kubernetes evolved, new resource types were organized into specific named groups to keep the API modular.

- **RBAC Syntax:** The group name as a string (e.g., `["apps"]`, `["networking.k8s.io"]`).
- **API URL Path:** `/apis/<group-name>/<version>` (e.g., `/apis/apps/v1`).
- **YAML `apiVersion`:** `<group-name>/<version>` (e.g., `apps/v1`).

---

### 3. Quick Reference Table

| API Group (for RBAC) | YAML `apiVersion` | Common Resources | Purpose / Scope |
|---|---|---|---|
| `""` *(Core)* | `v1` | `pods`, `services`, `secrets`, `configmaps`, `namespaces`, `nodes`, `persistentvolumeclaims` | Foundational primitives of Kubernetes |
| `"apps"` | `apps/v1` | `deployments`, `statefulsets`, `daemonsets`, `replicasets` | Application workload orchestration and scaling |
| `"batch"` | `batch/v1` | `jobs`, `cronjobs` | Ephemeral, run-to-completion tasks and scheduled work |
| `"networking.k8s.io"` | `networking.k8s.io/v1` | `ingresses`, `networkpolicies`, `ingressclasses` | External traffic routing and network security rules |
| `"storage.k8s.io"` | `storage.k8s.io/v1` | `storageclasses`, `volumeattachments`, `csinodes` | Dynamic storage provisioning and driver integration |
| `"rbac.authorization.k8s.io"` | `rbac.authorization.k8s.io/v1` | `roles`, `rolebindings`, `clusterroles`, `clusterrolebindings` | Identity access control and cluster permissions |
| `"autoscaling"` | `autoscaling/v2` | `horizontalpodautoscalers` (hpa) | Automated workload autoscaling |
| `"admissionregistration.k8s.io"` | `admissionregistration.k8s.io/v1` | `validatingwebhookconfigurations`, `mutatingwebhookconfigurations` | Custom policy interceptors and admission webhooks |
| `"policy"` | `policy/v1` | `poddisruptionbudgets` (pdb) | High-availability constraints during voluntary disruptions |

---

### 4. How to Find API Groups via CLI

You do not need to memorize every group. Inspect them directly from your cluster:

#### List all supported resources, their shortnames, and API groups:
```bash
kubectl api-resources
```


## RBAC in Kubernetes:

- RBAC stands for Role-Based Access Control. It is a way to control access to resources in Kubernetes based on the roles of users or service accounts. It allows us to define roles and assign them to users or service accounts, and then we can use those roles to control access to resources in Kubernetes.

### Role and RoleBinding:
- `role` is limited to a namespace.
- We can use `rb`(role binding)to bind a `role` to a user.
- If we wanna provide similar access to 10 users we can create a group and assign that `role` to that group.

### Role Yaml example
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: app-manager
rules:
  # 1. Core group resources (empty string "")
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps"]
    verbs: ["get", "list", "watch"]

  # 2. Named group: "apps"
  - apiGroups: ["apps"]
    resources: ["deployments", "statefulsets"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]

  # 3. Named group: "networking.k8s.io"
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list"]
```
- First rules means - giving get, list, watch access to pods, services and configmaps in core group.
- Second rules means - giving get, list, watch, create, update and patch access to deployments and statefulsets in apps group.
- Third rules means - giving get, list access to ingresses in networking.k8s.io group.
- In above one as we haven't mentioned the namespace - it will be created in the default namespace. And if we wanna create it in another namespace - we can mention that namespace in the metadata section.

#### Below is Different example
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-app
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "list"]
  resourceNames: ["myapp"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
  resourceNames: ["mydb"]
```
- In above we mentioned the exact `resourceNames` to make that specific rule for a specific resource. 
- First Rules means - giving get, create and list access to pods named `myapp` in core group.
- Second Rules means - giving list access to pods named `mydb` in core group.

#### Role Buinding Yaml example
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jane-developer-binding
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```
- In above `subject` is the one who gets the access, in this case User named `jane`.
- and `roleRef` is the role which is being assigned to the subject, in this case Role named `developer`.



### ClusterRole and ClusterRoleBinding:
- `clusterrole` is not limited to a namespace, it can be used to provide access to resources across the cluster.
- We can use `crb`(cluster role binding) to bind a `clusterrole` to a user or a group.
- We do bind `clusterrole` to a group of admin users with `clusterrolebinding` so that they can manage the cluster.

Example of clusterRole yaml file is as below:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: [""]
  resources: ["nodes"] # Similarly we can do for 'namespaces' as it is a cluster wide resource.
  verbs: ["get", "create", "list", "delete", "update"]
```

ClusterRoleBinding yaml file is as below:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: cluster-admins
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

### Interesting Note:
- We can also create a `clusterrole` for namespaced resources - like `pods`, `services`, `configmaps`, etc.

What it does (Depends on how you bind it)
The behavior depends entirely on whether you bind it with a `RoleBinding` or a `ClusterRoleBinding`:

1. Paired with a `RoleBinding` $\rightarrow$ Reusable Template (Scoped to 1 Namespace)
- What happens: The user only gets access to pods inside the specific namespace where the `RoleBinding` lives.
- Why do it: Instead of creating the exact same `Role` in 20 different namespaces, you define one `ClusterRole` once. Then, you place a lightweight `RoleBinding` in each namespace pointing to that single `ClusterRole`.

2. Paired with a `ClusterRoleBinding` $\rightarrow$ Cluster-Wide Access Across All Namespaces
- What happens: The user gets access to pods across every single namespace in the cluster, present and future (e.g., `default`, `kube-system`, `custom namespaces`).
- Why do it: For monitoring tools, log collectors, or cluster auditors who need to read pods everywhere without maintaining bindings in individual namespaces.

**So**
- We apply the above yaml files using the same `kubectl apply -f <file-name>.yaml` command.
- View them with `kubectl get roles`, `kubectl describe role developer`, `kubectl get rolebindings`, `kubectl describe rolebinding jane-developer-binding`, `kubectl get clusterroles`, `kubectl describe clusterrole cluster-admin`, `kubectl get clusterrolebindings`, `kubectl describe clusterrolebinding read-secrets-global` commands.
- 

### Users & Groups in Kubernetes:

- Kubernetes doesn't have a built-in user management system, so k8s admins use external sources for example Static Token file, Certificates, 3rd Party services like LDAP etc. to manage users and groups in Kubernetes.
- So `API Server` using the knowledge of external user management system - authenticates the users and groups, and then it uses RBAC to authorize the users and groups to access resources in Kubernetes. 
  - We can pass the users data ie - `users.csv` to api server like this `--token-auth-file=users.csv` and then api server will use that file to authenticate the users. And we can also use `--authorization-mode=RBAC` to enable RBAC in api server.

There are two types of users in Kubernetes:
- **Human Users** - This consists - k8s admins, developers, authorised by `clusterrole` and `role`.
- **Application Users** - This consists internal applications i.e. - Prometheus, Internal applications which needs data internally from another applications. And External application which needs access to the cluster ie - `Jenkins`, `Terraform` etc.

- So for Application users we create a `sa`(service account) which is a special type of user that is used by applications to access resources in Kubernetes. And we can use `role` and `clusterrole` to authorize the service accounts to access resources in Kubernetes. 
  - `sa` is created using - `kubectl create serviceaccount sa1`
