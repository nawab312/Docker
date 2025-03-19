**Docker**
Docker is an open-source platform that allows you to automate the deployment, scaling, and management of applications inside lightweight, portable containers. Containers bundle an application with all its dependencies, such as libraries, configurations, and binaries, ensuring that it runs consistently across different environments (e.g., development, testing, production).

**Dockerfile**
- A Dockerfile is a text file containing instructions to build a Docker image.
- It automates the process of creating Docker images by defining the application's environment, dependencies, and configurations.
- The order of instructions matters because each step creates a layer in the image.
- Using .dockerignore can prevent unnecessary files from being included in the image.

![image](https://github.com/user-attachments/assets/4013ea61-b384-4b94-b2de-a049e98e034d)

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



