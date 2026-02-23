# Docker – Deep Short Notes

- **Playlist**: Docker and Kubernetes Tutorial for Beginners – TechWorld with Nana [Link](https://youtube.com/playlist?list=PLy7NrYWoggjwPggqtFsI_zMAwvG0SqYCb&si=lBQuMIQ0CFECW_K5)
- Total Notes credits goes to TechWorld with Nana for the original content and explanations in the videos.

---
## 1. What Is Docker?

- Containerization vs traditional deployment: Traditional deployments install runtimes, libraries, and app directly on each server, while containerization puts everything the app needs into a self‑contained unit that runs the same on any host.  
- OS‑level virtualization: Docker uses Linux kernel features (namespaces, cgroups) to isolate processes so each container thinks it has its own filesystem, network, and resources without needing a separate OS.  
- Image as a blueprint: A Docker image is a versioned, read‑only snapshot of an application stack (base OS layer, runtime, app code, configs) that you can instantiate multiple times as containers.  
- Container as a running instance: A container is a live process created from an image with its own isolated environment, which can be started, stopped, removed, and recreated quickly.  
- Layered filesystem and caching: Each instruction in a Dockerfile adds a layer, and unchanged layers are reused across builds and pulls, making image distribution and rebuilding much faster in practice.  
- Registry and collaboration: Docker registries (like Docker Hub or private registries) act as central repositories so teams can share images, promote them through environments, and roll back to older tags.  
- Solving “works on my machine”: By embedding dependencies and configuration in the image, Docker ensures the same binary artifact is used in dev, test, and prod, drastically reducing environment drift issues.  

![Before Docker](./assets/before-docker.png)
![After Docker Containers](./assets/after-docker.png)
---

## 2. Docker Containers

- Difference from VMs: Unlike VMs that emulate hardware and run a full guest OS, containers are just processes on the host with isolated views of the system, which makes them much lighter and faster.  
- Single responsibility design: Typical practice is to run one main process (e.g., web server) per container, so each container is focused, easier to reason about, and simpler to scale independently.  
- Port mapping in practice: Using `-p 8080:80` maps host port 8080 to container port 80 so you can reach an NGINX or app server in the container from your browser via `http://localhost:8080`.  
- Stateless vs stateful containers: Stateless app containers are safe to delete and recreate freely, but stateful components (like databases) must use volumes so that data outlives container instances.  
- Multiple containers from one image: You can spin up many identical containers from the same image, which is the foundation for horizontal scaling and rolling out multiple app replicas.  
- Inspecting runtime isolation: Tools like `docker ps` and `docker top` show that processes are isolated per container but still visible on the host, illustrating how containers are “just processes” with boundaries.  

---

## 3. Installing Docker

- Docker Engine components: Installation typically brings the daemon (server), the CLI (`docker` client), and on desktop environments a UI that manages settings, resources, and Kubernetes integration.  
- Linux installation specifics: On Linux you usually configure the official Docker repository, install `docker-ce` packages, and then add your user to the `docker` group so you can run commands without `sudo`.  
- Windows & Mac via Docker Desktop: On non‑Linux systems Docker Desktop runs a lightweight Linux VM behind the scenes, bridging file sharing and networking so containers behave like local processes.  
- Post‑install verification: Running `docker version` checks client/daemon connectivity, and `docker run hello-world` pulls a small test image and prints a message if everything is configured correctly.  
- Common installation pitfalls: Typical issues include the daemon not starting on boot, missing group membership causing permission errors, and corporate proxies blocking image pulls from public registries.  

---

## 4. Core Docker Commands

- `docker pull <image:tag>`: Explicitly fetches a specific image and tag from a registry so subsequent runs don’t incur network overhead; helpful for pre‑warming environments or working offline.  
- `docker run -d -p 8080:80 --name web nginx`: Combines image pull, container creation, and startup into one command, runs it in the background, and exposes the container’s port 80 through host port 8080.  
- `docker ps` vs `docker ps -a`: `docker ps` shows only running containers, while `-a` reveals stopped ones too, which is essential when debugging why a container exited immediately after start.  
- `docker stop` vs `docker kill`: `stop` sends a graceful signal giving the process time to clean up, whereas `kill` terminates immediately, which you only use when the process is hung.  
- `docker logs <name>` and `docker logs -f`: The basic logs command shows past stdout/stderr, and `-f` follows new log lines, similar to `tail -f`, making it easier to watch live application behavior.  
- `docker rm <name>`: Removes the container metadata so you don’t accumulate dead containers, but does not touch volumes or images unless explicitly told to.  
- `docker rmi <image>`: Deletes one or more images from local storage, which is useful for reclaiming disk space when old image versions are no longer needed.  
- `docker system prune`: Cleans up dangling images, stopped containers, unused networks, and build cache, which is a typical step in keeping dev machines from filling up.  

---

## 5. Debugging Containers

- Using logs as first stop: `docker logs` quickly reveals application‑level errors (stack traces, bad configs, missing env vars) without needing to attach into the container interactively.  
- Live log streaming for repro: `docker logs -f` is used while reproducing a bug so you can correlate your actions (HTTP requests, CLI inputs) with what the application prints in real time.  
- Inspecting exit codes: When a container exits immediately, checking `docker ps -a` and `docker inspect` helps you see the exit code and reason (e.g., misconfigured command) to know where to look next.  
- `docker exec -it <container> sh/bash`: Opens a shell into the running container, letting you inspect environment variables, installed packages, open ports, and file structure exactly as the app sees them.  
- Investigating config problems: Inside the container you can check config files, test network connectivity (e.g., `curl` to another service), and verify that mount points or secrets are actually present.  
- Combining inspect and exec: A common pattern is to `inspect` to understand mounts, env vars, and working directory, and then `exec` in to verify those settings empirically, closing the loop on configuration issues.  

---

## 6. Docker vs Virtual Machines

- Isolation model difference: VMs virtualize hardware and run an entire guest OS per instance, while containers virtualize at the OS level, isolating processes but reusing the host kernel.  
- Resource overhead: Each VM carries OS memory and background processes, so a host can run fewer VMs than containers for the same hardware budget.  
- Boot and deployment times: VM boot involves OS startup and services, often taking tens of seconds or minutes, whereas containers usually start in under a second because they just launch a process.  
- Image vs VM template: Docker images are typically smaller and more modular than VM images; layering and registry distribution make them easier to transport and version.  
- Use cases: VMs are great for strong isolation or running different OS types, while containers are ideal for microservices, CI pipelines, and high‑density stateless workloads.  
- Combined usage: In many real setups, clusters run on VMs or cloud instances, and containers are scheduled on top by an orchestrator like Kubernetes, blending both technologies.  

![docker-vs-vm](./assets/docker-vs-vm.png)
---

## 7. Docker Workflow

- Dev build cycle: Developers write code and a Dockerfile, build an image locally (`docker build`), and run containers to verify behavior; Docker becomes part of the inner dev loop.  
- CI pipeline integration: CI servers build images on each commit, run tests inside containers to guarantee consistent environments, and push versioned images to registries on success.  
- Promotion across environments: The same image tag (e.g., `my-app:1.2.3`) is pulled in dev, staging, and production, ensuring you deploy exactly what was tested.  
- Tagging strategy: Semantic versioning or Git‑SHA tags allow precise rollbacks and reproducible deployments, while `latest` is used only as a convenience in non‑critical setups.  
- Automated deployment: Orchestrators like Docker Compose, Swarm, or Kubernetes pull images and create containers per declarative configs, decoupling the build pipeline from runtime infrastructure.  
- Observability and debugging: Logs and metrics are collected at container level, enabling per‑service visibility in a microservices architecture, which is much harder with bare processes.  

---

## 8. Developing with Docker

- Local development via bind mounts: Mapping your source directory into the container (`-v ./src:/app`) lets you edit files on your machine while the container uses them directly, often combined with hot‑reload tools.  
- Dev images with tooling: Development‑oriented images often include compilers, debuggers, and package managers that are omitted in production images to keep them slim and secure.  
- Reproducible dev environments: Onboarding becomes easier because new developers only need Docker installed; running a single compose command can bring up the full stack with the correct versions.  
- Running dependencies locally: Databases, message brokers, and caches are run as containers, so you don’t pollute your host OS and can spin them up or down as needed for different projects.  
- Parity with production: By using the same base image and runtime as in production, you reduce “dev vs prod” differences and catch issues earlier in the development cycle.  

---

## 9. Docker Compose

- Multi‑service definition: `docker-compose.yml` describes all services (web, db, cache) plus their ports, volumes, environment, and dependencies in a single versioned file.  
- Automatic network creation: Compose creates a network for the project so services can talk to each other using service names as DNS, avoiding hard‑coded IPs.  
- Docker network concept (important): When you run multiple services in the same `docker-compose.yml`, Docker Compose automatically creates and manages a single network for that project, and connects all services to it by default.
- Container-to-container communication: From inside one container, you can reach another container by using the **service name** as the hostname (e.g., `mongodb`), because Compose provides internal DNS on that network.
  - Example: From `my-app` container, connect to MongoDB using hostname `mongodb` (not `localhost`).

```js
// use when starting application as docker container
let mongoUrlDocker = "mongodb://admin:password@mongodb";
```
- Service dependencies: The `depends_on` property expresses startup ordering (e.g., app depends on DB), making `docker-compose up` start services in a sensible sequence.  
- Easy stack lifecycle: `docker-compose up -d` brings up the full stack in the background, and `docker-compose down` tears everything down (optionally keeping volumes for data).  
- Scaling with Compose: You can scale a service (`docker-compose up --scale web=3`) to simulate multiple replicas behind the same service name, giving a feel for horizontal scaling.  
- Environment isolation: Different compose files or overrides let you simulate dev vs test settings (different environment variables, ports, or resource limits) without changing core definitions.  
- Simple example (app + MongoDB + mongo-express):

```yml
version: '3'
services:
  my-app:
    image: 65458435491.sed.ecr.eu-north-1.amazonaws.com/my-app:1.2
    ports:
      - 3000:3000

  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb

volumes:
  mongo-data:
```

---

## 10. Dockerfiles

- Base image selection: `FROM` chooses a base such as `node:alpine` or `python:3.9`, which brings an OS and runtime stack that influences image size and security.  
- Build context and `COPY`: The build context is the directory sent to Docker during `docker build`; `COPY` takes files from that context into the image, so controlling context size is critical.  
- Layer optimization: Combining package installs into a single `RUN` step and cleaning up caches can significantly reduce the final image size and build time.  
- `EXPOSE` semantics: `EXPOSE 80` documents which port the container is expected to listen on and is used by some tools, though it doesn’t actually publish the port to the host by itself.  
- `CMD` vs `ENTRYPOINT`: `ENTRYPOINT` defines the main executable, while `CMD` supplies default arguments; understanding their interaction is key to creating flexible images.  
- Multistage builds: Using an initial “builder” stage for compiling assets and a second “runtime” stage that only copies the compiled output yields small, production‑ready images without dev tools.  
- `.dockerignore`: Listing paths like `node_modules`, `.git`, and build artifacts prevents them from being sent as build context, saving time and space.  
- Simple example (Node.js app Dockerfile):

```dockerfile
FROM node:13-alpine

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

RUN mkdir -p /home/app

COPY ./app /home/app

# set default dir so that next commands executes in /home/app dir
WORKDIR /home/app

# will execute npm install in /home/app because of WORKDIR
RUN npm install

# no need for /home/app/server.js because of WORKDIR
CMD ["node", "server.js"]
```

---

## 11. Private Registries (AWS ECR)

- Need for private registries: Organizations use private registries to store proprietary or internal images, enforcing access control instead of relying on public Docker Hub.  
- AWS ECR example: Amazon Elastic Container Registry is a managed registry where you push images using AWS credentials and pull them from AWS compute services like ECS or EKS.  
- Authentication flow: CLI tools retrieve a short‑lived login token and pass it to `docker login`, after which `docker push` and `docker pull` work against the private endpoint.  
- Image naming conventions: Private images include registry URL prefixes (e.g., `123456789012.dkr.ecr.region.amazonaws.com/my-app:tag`) so Docker knows where to fetch them from.  
	- Example: `65458435491.sed.ecr.eu-north-1.amazonaws.com/my-app:1.2`
- Integrating with CI/CD: Pipelines authenticate to the registry, build images, push them, and then deployment jobs pull those images into target clusters, enforcing a clear, auditable path from source to runtime.  

---

## 12. Deploying with Docker Compose

- Single‑host deployment pattern: Docker Compose can be used on a server to run a multi‑container app stack without needing a full orchestrator like Kubernetes or Swarm.  
- Using named volumes for persistence: Databases and other stateful services in Compose use named volumes so their data survives container recreation and stack restarts.  
- Mapping external ports: `ports` in the compose file expose services to the outside world (e.g., mapping app port 80 to host port 80 for user traffic).  
- Environment variables and secrets: Compose lets you define environment variables per service and load them from `.env` files to keep configuration separate from images.  
- Restart policies for resilience: `restart: always` or `on-failure` in compose ensures containers automatically come back if they crash or if the host reboots.  
- Practical example: setting `restart: always` on `mongo-express` prevents it from failing when `mongodb` isn’t ready yet (it keeps retrying).  
- Environment‑specific overrides: Extra compose files (e.g., `docker-compose.prod.yml`) tweak settings like resource limits or volume locations for production without changing base definitions.  

---

## 13. Docker Volumes

- Container filesystem ephemerality: By default, data written inside a container’s filesystem disappears when the container is removed, which is unsafe for important data.  
- Volume purpose: Volumes provide a separate, persistent storage area managed by Docker, allowing containers to be replaced while the data stays intact.  
- 3 types of Docker volume mounts (with `docker run -v`):
	- **Host volumes (bind mounts)**: You decide the exact location on the host where the data should live.
		- Syntax: `docker run -v host_directory:container_directory`
		- Example:
			- `docker run -v /home/mount/data:/var/lib/mysql/data`
	- **Anonymous volumes**: You pass only the container path; Docker automatically creates a (randomly named) volume on the host.
		- Syntax: `docker run -v container_directory`
		- Example:
			- `docker run -v /var/lib/mysql/data`
	- **Named volumes**: You give the volume a name and also pass the container path; Docker creates/manages the volume for you.
		- Syntax: `docker run -v name:container_directory`
		- Example:
			- `docker run -v name:/var/lib/mysql/data`
		- This is the most commonly used approach and generally the recommended way to manage persistent data.
- Named volume in Compose looks like: `mongo-data:/data/db` (where `mongo-data` is created/managed by Docker).
- Database use case: Tutorials commonly show a database container writing its data directory to a volume so that upgrades or redeploys of the DB container do not wipe the stored data.  
![docker-volumes.png](./assets/docker-volumes.png)
---

## 14. Volumes in Practice

- Creating a volume in commands: The demo shows creating a named volume via CLI or compose and then attaching it to a container’s data directory (e.g., `/var/lib/mysql`).  
- Demonstrating persistence: After writing data to the service, the container is removed and recreated with the same volume, and the data is still there, proving the decoupling between container and storage.  
- Inspecting volume details: `docker volume ls` and `docker volume inspect` reveal internal mount paths and metadata, which is helpful when debugging storage configurations.  
- Cleaning unused volumes: The demo illustrates that volumes are not removed automatically with containers, so you must explicitly delete them or run `docker volume prune` to avoid orphaned storage. 