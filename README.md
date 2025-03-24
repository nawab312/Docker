### Docker Registry & Image Management ###
- **Pushing Images to Docker Hub (docker push)**
- **Running a Private Docker Registry (docker run registry)**
- **Tagging Docker Images (docker tag)**
- https://github.com/nawab312/Docker/blob/main/Docker_Registry_Image_Management.md


**Docker**

Docker is an open-source platform that allows you to automate the deployment, scaling, and management of applications inside lightweight, portable containers. Containers bundle an application with all its dependencies, such as libraries, configurations, and binaries, ensuring that it runs consistently across different environments (e.g., development, testing, production).

**Dockerfile**
- A Dockerfile is a text file containing instructions to build a Docker image.
- It automates the process of creating Docker images by defining the application's environment, dependencies, and configurations.
- The order of instructions matters because each step creates a layer in the image.
- Using .dockerignore can prevent unnecessary files from being included in the image.

![image](https://github.com/user-attachments/assets/4013ea61-b384-4b94-b2de-a049e98e034d)

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



