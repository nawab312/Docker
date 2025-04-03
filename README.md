### Introduction to Docker ###

**What is Docker?**

Docker is an open-source platform that allows you to automate the deployment, scaling, and management of applications inside lightweight, portable containers. Containers bundle an application with all its dependencies, such as libraries, configurations, and binaries, ensuring that it runs consistently across different environments (e.g., development, testing, production).

**Benefits of using Docker**
- *Portability*
  - Docker containers run the same way on any system (laptop, server, cloud).
  - Avoids the “Works on my machine” problem.
  - Example: A containerized app can run on AWS, Azure, GCP, or on-premises without changes.
- *Faster Application Deployment & Scaling*
  - Containers start in seconds, unlike VMs which take minutes.
  - Quickly scale applications up or down using Docker Compose, Swarm, or Kubernetes.
- *Efficient Use of System Resources*
  - Lightweight: Uses less CPU, memory, and disk compared to VMs.
  - Containers share the same OS kernel, reducing overhead.
  ![image](https://github.com/user-attachments/assets/df7518ed-626e-48be-ad3a-96189cec1a5a)

**Docker vs Virtual Machines (VMs)**

Docker and Virtual Machines (VMs) are both used for running applications in isolated environments, but they have key differences in architecture, performance, and resource utilization.
![image](https://github.com/user-attachments/assets/58339964-a7f6-4cdb-a16a-df9bc42e3d63)

### Docker Images ###
An **Alpine Image** is a lightweight, security-focused, and minimal Linux distribution designed for containers. It is based on Alpine Linux, which is known for being small, fast, and optimized for security. Key Features of Alpine Images:
- Extremely Small Size – Typically ~5MB compared to Debian-based images (~25MB+).
- Security Focused – Uses *musl libc* and *busybox*, reducing vulnerabilities.
- Uses `apk` for Package Management – Alpine Package Keeper (`apk add package-name`)
When to Use Alpine?
- If you need a small, lightweight image (e.g., microservices, cloud deployments).
- If your app is fully compatible with `musl libc` (some software needs `glibc`, which Alpine lacks)
When Not to Use Alpine?
- If your application depends on `glibc-based` libraries (e.g., some Python ML libraries).


### Dockerfile (Building Custom Images) ###
**Dockerfile**
- A Dockerfile is a text file containing instructions to build a Docker image.
- It automates the process of creating Docker images by defining the application's environment, dependencies, and configurations.
- The order of instructions matters because each step creates a layer in the image.
- Using .dockerignore can prevent unnecessary files from being included in the image.

![image](https://github.com/user-attachments/assets/4013ea61-b384-4b94-b2de-a049e98e034d)

**CMD and ENTRYPOINT**
- Both CMD and ENTRYPOINT define what command should run when a container starts
- *CMD* - Default Command. Specifies a default command for the container, but it can be overridden when running the container
  ```dockerfile
  FROM ubuntu:latest
  CMD ["echo", "Hello World"]
  ```
  ```bash
  docker build -t prac .
  docker run cmd_1 
  Hello, World!
  ```
  ```bash
  docker run prac ls
  bin
  boot
  dev
  etc
  home
  ```
  - It overrides CMD and executes `ls` instead.
- *ENTRYPOINT* - Mandatory Command. Defines a command that always executes, even if arguments are passed during docker run. It is mandatory in scenarios where the container should always execute a specific command, and users should not be able to override it at runtime.
  ```dockerfile
  FROM ubuntu:latest
  ENTRYPOINT ["echo", "Hello World"]
  ```
  ```bash
  docker build -t prac_1 .
  docker run prac_1 
  Hello, World!
  ```
  ```bash
  docker run prac_1 ls
  Hello, World! ls
  ```
  - Unlike CMD, the Command cannot be Overridden, but Arguments are appended.
  - Override ENTRYPOINT
    ```bash
    docker run --entrypoint ls prac_1
    bin
    boot
    dev
    etc
    home
    ```

**ADD and COPY**
- Both ADD and COPY are used to copy files and directories from the host system to the Docker image. However, they have key differences in how they behave.
  - *COPY* – Simple File Copying. COPY is a Straightforward Command that copies files or directories from the Build Context (Local System) into the container. It does not extract compressed files or support remote URLs.
  - *ADD* – Advanced Copying. ADD does everything COPY does but with extra features:
    - It automatically extracts .tar, .tar.gz, and .zip files.
    - It supports remote URLs, downloading files directly from the Internet.

The **docker build** command is used to create a Docker image from a Dockerfile. It processes the instructions in the Dockerfile step by step and generates a reusable image.
 ```bash
docker build [OPTIONS] -t IMAGE_NAME[:TAG] PATH
```
```bash
docker build -t my-python-app .
```
This will create an image named `my-python-app` from the Dockerfile in the current directory (.).

**Mutli-Stage Dockerfile**
- A multi-stage Dockerfile helps build lightweight and optimized Docker images by using multiple stages in a single Dockerfile. It allows you to:
  - Reduce image size
  - Improve security by removing unnecessary dependencies
  - Keep the build and runtime environments separate

```dockerfile
# Stage 1: Build dependencies
FROM python:3.9 AS builder
WORKDIR /app

# Install dependencies in a virtual environment
COPY requirements.txt .
RUN python -m venv /venv && \
    /venv/bin/pip install --no-cache-dir -r requirements.txt

# Stage 2: Final runtime image
FROM python:3.9-alpine
WORKDIR /app

# Copy only the virtual environment from the builder stage
COPY --from=builder /venv /venv
COPY . .

# Set the correct PATH to the virtual environment
ENV PATH="/venv/bin:$PATH"

# Run the Flask application (assuming `app.py` is the entry point)
CMD ["python", "app.py"]
```
- If we did this `COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages`. Then we are copying the whole site-packages directory will result in a larger image than needed, as it could include compiled C extensions and other unnecessary files that were needed only during the build phase. To avoid this, we might want to consider using a virtual environment in the builder stage.

The **docker commit** command is used to create a new Docker image from a running or stopped container. This allows you to save changes made inside a container and reuse them later as a new image.
```bash
docker commit [OPTIONS] CONTAINER_ID NEW_IMAGE_NAME[:TAG]
```
Options:
- `-a "author"` → Add author metadata
- `-m "message"` → Add a commit message
- `-c "CMD command"` → Override default CMD instruction
When to Use docker commit?
- Capturing manual changes made inside a running container
- Creating a base image after debugging or modifying a container
- Saving a container state before destroying it






### If Docker Containers are Consuming Too Much Disk How to Fix it ###
**Check Docker Disk Usage**
- This will show the space used by images, containers, volumes, and build cache.
  ```bash
  docker system df
  TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
  Images          27        27        9.806GB   2.714GB (27%)
  Containers      54        0         1.798GB   1.798GB (100%)
  Local Volumes   23        6         4.114GB   2.74GB (66%)
  Build Cache     230       0         4.003GB   4.003GB
  ```

**Remove Unused Docker Resources**
- Remove Unused Containers
  - List all stopped containers: `docker ps -a`
  - Remove all stopped containers: `docker container prune -f`
- Remove Unused Images
  - List all docker Images: `docker images`
  - Remove unuse Images: `docker image prune -a -f`
- Remove Unused Volumes
  - List all Volumes: `docker volume ls`
  - Remove Dangling(Unused) Volumes: `docker volume prune -f`
  - If you want to Delete all Volumes: `docker volume rm $(docker volume ls -q)
- Remove Build Cache
  - Clear the Build Cache: `docker builder prune -a -f`
 
**Clean Everything at Once**
- If you want to remove everything (stopped containers, unused networks, images, and build cache), use:
  ```bash
  docker prune system -a -f
  ```

**Limit Docker Log Size**
- Docker container logs can grow large. To limit log size, add the following to `/etc/docker/daemon.json`:
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```
- Then restart Docker: `systemctl restart docker`

**Move Docker Storage to Another Disk**
- If your `/var/lib/docker` directory is consuming too much space, you can move it to another disk:
  - Stop Docker: `systemctl stop docker`
  - Move the Data: `mv /var/lib/docker /new/path/docker`
  - Create a Symlink: `ln -s /new/path/docker /var/lib/docker`
  - Restart Docker: `systemctl restart docker`
 
**Regular Maintenance**
- Set up a cron job to automatically clean up Docker resources periodically:
  ```bash
  0 3 * * * docker system prune -a -f
  ```
- This will clean unused Docker resources daily at 3 AM.

---

### Securely Storing and Distributing Docker Images Using DockerHub ###

At my previous project, we were using DockerHub to store and distribute container images for our microservices architecture. However, we faced security concerns related to:
- Public Exposure of Images: Initially, some images were accidentally pushed to a public repository, making them accessible to unauthorized users.
- Managing Access Control: Multiple developers needed access, but we needed to restrict who could push and pull images securely.
- Securing Secrets in Docker Images: Some application images contained sensitive credentials (like API keys) embedded in `Dockerfile`, posing a risk if leaked.

**Solution**
- **Restricting Public Access**
  - We converted all repositories to private to prevent unauthorized access.
  - Used *DockerHub Teams & Organizations* to grant controlled access.
- **Using IAM-Based Authentication for Secure Access**
  - Instead of sharing DockerHub credentials, we implemented *personal access tokens (PAT)* for authentication.
  - Developers used `docker login` with PATs instead of storing passwords in CI/CD pipelines.
  ```bash
  docker login -u myuser -p mytoken my-private-registry.com
  docker push my-private-registry.com/myapp:latest
  ```
- **Implementing Image Signing & Verification**
  - We used *Docker Content Trust (DCT)* to sign images before pushing to DockerHub.
  - This ensured that only trusted images were deployed in production.
  ```bash
  export DOCKER_CONTENT_TRUST=1
  docker push myrepo/myimage:latest
  ```
- **Removing Hardcoded Secrets from Docker Images**
  - Used AWS Secrets Manager & Environment Variables instead of storing secrets inside Dockerfile.
  - Bad Practice (Dockerfile with secrets embedded)
  ```bash
  ENV DB_PASSWORD=mysecretpassword
  ```
  - Secure Approach (Using AWS Secrets Manager in an Entrypoint Script)
  ```bash
  DB_PASSWORD=$(aws secretsmanager get-secret-value --secret-id my-db-secret --query SecretString --output text)
  export DB_PASSWORD
  ```
- **Used DockerHub’s Image Scanning to detect vulnerabilities before pushing images.**
```bash
trivy image myrepo/myimage:latest
```



