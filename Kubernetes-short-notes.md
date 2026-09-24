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


### Deployment and Service YAML file:

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

### Namespaces:

There are two namespaces:
1. **Namespace in Linux** - It is a feature of the Linux kernel that allows you to create isolated environments for processes. Each namespace has its own set of resources, such as process IDs, network interfaces, and file systems. This allows you to run multiple instances of the same application on the same machine without them interfering with each other. Example - A docker container gets its own namespace for process IDs, network interfaces, and file systems, so that it can run independently of other containers on the same machine.
   - Why it is needed - It is needed to provide isolation between different applications running on the same machine. Without namespaces, all processes would share the same resources, which could lead to conflicts and security issues. By using namespaces, you can ensure that each application has its own isolated environment, which improves security and stability.
2. **Namespace in Kubernetes** - It is a way to divide cluster resources between multiple users.
### Why we use namespaces

 1. **Hard to manage:** Everything in one default namespace will be messy and hard to manage. Instead of that we can one namespace for `database`, one for `monitoring`, `elastic stack`, `nginx-ingress` and so on. This will make it easier to manage and organize resources in the cluster.
 2. **Name Conflicts in teams:** Team A had a deployement with abc.yaml name, later Team B did kubectl apply -f abc.yaml which will cause a conflict.
 3. **Resources Sharing:** 
    1. We can use same resources i.e. - `nginx-ingress controller` , `elastic stack` for both staging and production namespaces. This will save resources and cost.
    2. Blue green deployment - We can use two namespaces for blue and green deployments. This will allow us to deploy new versions of our application without affecting the existing version. We can switch between the two versions by changing the service selector to point to the new version.
 4. **Resources Quotas:** We can set resource quotas for each namespace, which limits the amount of CPU, memory, and storage that can be used by the resources in that namespace. This helps to prevent one team from consuming all the resources in the cluster and ensures that resources are allocated fairly among different teams.

### Characteristics of Namespaces in Kubernetes:
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

### Different types of Namespaces in Kubernetes:
1. **Default Namespaces** - These are the namespaces that are created by default when you create a Kubernetes cluster. They include:
- `default` - This is the default namespace for resources that are not assigned to any other namespace.
- `kube-system` - This namespace is used for resources that are managed by the Kubernetes system, such as the API server, scheduler, and controller manager.
- `kube-public` - This namespace is used for resources that are publicly accessible, such as the Kubernetes dashboard.
- `kube-node-lease` - This namespace is used for resources that are related to node leases, which are used to track the availability of nodes in the cluster.
2. **User-defined namespaces** - These are the namespaces that you can create to organize your resources in a way that makes sense for your application. You can create as many user-defined namespaces as you need, and you can assign resources to them using labels and selectors.

### IP addresses for nodes and pods
- Each node gets a range of IP addresses for eg. 
  - Node 1 - 10.1.1.x
  - Node 2 - 10.1.2.x
- And each pod gets an IP from the range in which node it is running.
  - For example the pod running on node 1 can have `10.1.1.5` as ip address.

### Services


#### ClusterIP Service:

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

### Headless Service:
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

### NodePort Service:
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


### LoadBalancer Service:

- This is same to NodePort service - its just that the port opened by Loadbalancer is only accessible from the cloud provider's load balancer. So we can use this type of service in production environment. Rest of the things are same as NodePort service.
- And creating this LoadBalancer will automatically create a NodePort, ClusterIP as well.
- In this we pass `type: LoadBalancer` in the yaml file to make it a loadbalancer service. Otherwise it will be a clusterIP service by default.

### Ingress:
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


### Configuring Ingress in Minikube:
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