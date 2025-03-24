**Dockerfile**
- A Dockerfile is a text file containing instructions to build a Docker image.
- It automates the process of creating Docker images by defining the application's environment, dependencies, and configurations.
- The order of instructions matters because each step creates a layer in the image.
- Using .dockerignore can prevent unnecessary files from being included in the image.


![image](https://github.com/user-attachments/assets/4013ea61-b384-4b94-b2de-a049e98e034d)

The **docker build** command is used to create a Docker image from a Dockerfile. It processes the instructions in the Dockerfile step by step and generates a reusable image.
```bash
docker build [OPTIONS] -t IMAGE_NAME[:TAG] PATH
```
```bash
docker build -t my-python-app .
```
This will create an image named `my-python-app` from the Dockerfile in the current directory (.).
