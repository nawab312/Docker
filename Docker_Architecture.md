Docker follows a client-server architecture that enables containerized application development, packaging, and deployment. It consists of several key components:
- Docker Client
- Docker Daemon (Server)
- Docker Registry
- Docker Objects (Images, Containers, Volumes, Networks)

![image](https://github.com/user-attachments/assets/b524697e-49b9-41ae-8f0e-f6d95459b2d5)


**Docker Client**
- The Docker Client (`docker CLI`) is the primary way users interact with Docker.
- It sends commands (e.g., `docker build`, `docker run`, `docker ps`) to the Docker Daemon using REST API calls over a UNIX socket or TCP.

Example Command:
```bash
docker run -d -p 80:80 nginx
```
This command tells the Docker Daemon to:
- Pull the `nginx` image (if not already available).
- Create and start a container.
- Bind container port 80 to host port 80.

**Docker Daemon (Server)**
- The Docker Daemon (`dockerd`) is the core component that manages Docker objects.
- It listens for API requests and executes commands like creating, running, and monitoring containers.
- It interacts with
  - Container runtime (e.g., `
  - containerd`, `runc`) to start/stop containers.
  - Docker Registry to fetch/store container images.
  - Storage & Networking components for data persistence and connectivity.
 
Daemon Responsibilities
- Image management (pull, build, push)
- Container lifecycle (create, run, stop, delete)
- Network management (bridge, overlay, etc.)
- Volume and storage management

**Docker Registry**

A Docker Registry stores and distributes Docker images. The two main types are:
- Public Registry: Docker Hub (`hub.docker.com`)
- Private Registry: AWS ECR, Google Container Registry, Harbor

**Docker Objects**

Docker operates using various objects, including:
- A. Docker Images
- B. Docker Containers
- C. Docker Volumes
- D. Docker Networks

**Underlying Components***

Docker uses low-level tools to manage containers:
- Container Runtime
  - containerd: Responsible for running container processes.
  - runc: Executes containers as Linux processes.
