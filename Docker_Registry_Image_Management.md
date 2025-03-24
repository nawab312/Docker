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
