# OPERATING SYSTEMS 3

# EIOSY3A

**DOCKER PRIMER**

## INTRODUCTION TO DOCKER

Docker is a platform that simplifies the process of developing, shipping, and running applications using containers. Containers are lightweight, portable, and self-contained units of software that package an application along with its dependencies, such as libraries and configurations, ensuring it runs consistently across different computing environments. Unlike virtual machines, which emulate an entire operating system, containers share the host OS kernel, making them much more efficient in terms of resource usage and performance. Containers are lightweight, portable, and self-sufficient units of software that package everything an application needs to run.

This tutorial will guide students through the essential Docker concepts and commands with practical examples to help them get started. In this tutorial, we are to assume the environment you will be running the labs is Linux.

## 1. SETTING UP DOCKER

### Installation

For Debian/Ubuntu:
```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

For RHEL/CentOS:
```bash
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

### Starting Docker

Once installed, manage the Docker service:

Start Docker:
```bash
$ sudo systemctl start docker
```

Enable Docker to start on boot:
```bash
$ sudo systemctl enable docker
```

Check the status of Docker:
```bash
$ sudo systemctl status docker
```

## 2. WORKING WITH DOCKER IMAGES

Docker images are essentially blueprints for containers. Think of them like templates from which containers are created.

- Listing downloaded images:
```bash
$ docker images
# Lists all Docker images currently stored on your local machine
```

Example Output:
```
REPOSITORY TAG IMAGE ID CREATED SIZE
nginx latest 605c77e22612 2 weeks ago 133MB
ubuntu 20.04 ba6acccedd29 3 weeks ago 72.8MB
```

- Download an image from Docker Hub:
```bash
$ docker pull <image-name>:<tag>
# Downloads a Docker image from Docker Hub or a specified registry
# Fetches the image without creating a container
# :tag specifies the exact version (defaults to :latest if omitted)
```

Example:
```bash
# Pull the latest Nginx image
$ docker pull nginx:latest

# Pull a specific version of Ubuntu
$ docker pull ubuntu:20.04
```

- Removing An Image:
```bash
$ docker rmi <image-id>
# Removes a Docker image from the local machine
# Frees up disk space
# Removes unused or outdated images
# Cannot remove an image if it's being used by a container
```

Examples:
```bash
# Remove an image by its ID
$ docker rmi nginx

# Force remove an image (even if in use)
$ docker rmi -f nginx
```

## 3. RUNNING CONTAINERS

Containers are instances of images. Here's how to manage them:

- Run a Container Interactively:
```bash
$ docker run -it <image-name> <command>
```

- `i` - Stands for "interactive"
  - Keeps STDIN (standard input) open even if not attached
  - Allows you to interact with the container's process
  - Enables you to send input to the running container

- `t` - Stands for "tty" (Teletype)
  - Allocates a pseudo-terminal (terminal emulator)
  - Provides a terminal-like experience inside the container

Examples:
```bash
$ docker run ubuntu echo "Hello"
# Without -i, input is blocked
# This works, but no interaction

$ docker run -i ubuntu /bin/bash
# With -i, you can interact with processes
# Opens a bash prompt you can type into
```

**N.B** Don't run background processes using these options, such a web server or database server, rather use the -d option (explained in the next section)

- Run a Container In Detached Mode:
```bash
$ docker run -d <image-name>
# -d stands for "detached"
# Runs a container in the background
# Container runs without blocking your current terminal
# Ideal for long-running services and background processes
```

Detached mode characteristics:

*Terminal Independence*
- The container runs independently of the current terminal session
- You can continue using the same terminal for other commands
- Container continues running even if you close the terminal

*Use Cases*
- Web servers
- Databases
- Background processing services
- Continuous running applications

Examples:
```bash
$ docker run -d nginx
# Run Nginx web server in background

$ docker run -d -e MYSQL_ROOT_PASSWORD=mypassword mysql
# Run MySQL database in detached mode
```

- Map container ports to the host:
```bash
$ docker run -p <host-port>:<container-port> <image-name>
# <host-port>: The port on the host machine you want to use to access the application.
# <container-port>: The port inside the container that the application is listening on.
# This maps the host port to the container's port.
# <image-name>: The name of the Docker image to run.
```

Examples:
```bash
# MySQL container on port 3306
$ docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret mysql:5.7

# Nginx with HTTP and HTTPS ports mapped
$ docker run -p 8080:80 -p 8443:443 nginx
```

### Setting Environment Variables:

Sometimes when starting a container it might be a good idea to pass some environmental variables during creation time.

```bash
$ docker run -v app_data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=yourpassword mysql:5.7
# MYSQL_ROOT_PASSWORD: Sets the root user password (required for first-time initialization).
# Additional variables like MYSQL_DATABASE, MYSQL_USER, and MYSQL_PASSWORD can be used to create a database and user automatically.
```

## 4. MANAGING CONTAINERS

Docker command provides many subset commands to control all running containers:

```bash
$ docker ps -a
# List all containers (including stopped)

$ docker start <container-id or container-name>
# Start a stopped container

$ docker stop <container-id or container-name>
# Stop a running container

$ docker rm <container-id or container-name>
# Remove a container

$ docker container prune
# Remove all stopped containers

$ docker image prune
# Remove all unused images

$ docker system prune -a
# Remove everything unused

$ docker run -it --rm ubuntu /bin/bash
# Creates an interactive container that automatically removes itself when exited

$ docker restart <container-id or container-name>
# Restart a running container
```

## DOCKER VOLUMES

Containers are ephemeral by design, meaning their file system is destroyed when the container stops or is removed. Volumes allow you to store data outside the container's life-cycle so that it persists even after the container is deleted.

```bash
$ docker run -v <volume-name>:<container-path> <image-name>
```

Example:
```bash
$ docker run -v mysql_data:/var/lib/mysql
# mysql_data: Named volume created by Docker to store data persistently
# /var/lib/mysql: Directory inside the container where MySQL stores database files
```

Advantages of creating a persistent volume:
1. **Persistent Data**: Ensures database files remain available even after container deletion
2. **Re-usability**: Volumes can be reused by multiple containers

Managing volumes:
```bash
$ docker volume ls
# List all available volumes

$ docker volume rm <volume-name>
# Deletes a volume
```

## 6. NETWORKING IN DOCKER

Docker container networking determines how containers communicate with each other, the host system, and external networks.

### How Docker Networking Works
- Each container runs in its own isolated network namespace
- Has a private network interface
- Assigned a Docker IP address
- Has its own routing table and ports

### Docker Networking Drivers

#### Bridge Network (Default)
- Default networking for standalone containers
- Containers connected to a virtual bridge network
- Private IP addresses (e.g., 172.17.x.x)
- Traffic routed using port mapping

```bash
$ docker network create my_bridge
$ docker run --name container1 --network my_bridge nginx
$ docker run --name container2 --network my_bridge alpine ping container1
```

#### Host Network
- Shares host's network stack directly
- No private IP
- No port mapping required
- Best for high-performance scenarios

```bash
$ docker run --network host nginx
```

#### None Network
- Completely isolated containers
- No network access except loopback interface
- Useful for security-sensitive applications

```bash
$ docker run --network none alpine
```

#### Overlay Network
- Multi-host networking for Docker Swarm or Kubernetes
- Allows containers on different hosts to communicate
- Requires key-value store for metadata

```bash
$ docker network create --driver overlay my_overlay
$ docker service create --network my_overlay nginx
```

#### Macvlan Network
- Assigns containers their own MAC addresses
- Direct access to host network
- Useful for legacy systems or device integration

```bash
$ docker network create -d macvlan \
    --subnet=192.168.1.0/24 \
    --gateway=192.168.1.1 \
    -o parent=eth0 macvlan_net
$ docker run --network macvlan_net nginx
```

### Communication Between Containers

Same Network:
```bash
$ docker run --name app --network my_bridge alpine ping db
```

Different Networks:
```bash
$ docker network connect my_bridge container1
```

### DNS in Docker Networks
- Internal DNS server allows containers to resolve each other's names
- Customize container names with network aliases

```bash
$ docker run --network my_bridge --network-alias webserver nginx
```

Managing Networks:
```bash
$ docker network ls
# List networks

$ docker network inspect <network-name>
# Get detailed network information

$ docker network rm <network-name>
# Remove a network
```

## 7. BUILDING YOUR OWN DOCKER CONTAINER IMAGES

### Overview
A Dockerfile is a text file containing instructions for building a Docker image. It automates image creation, ensuring reproducibility and scalability.

### Why Use a Dockerfile?
- Automation and Reproducibility
- Customization
- Version Control
- Infrastructure-as-Code

### Anatomy of a Dockerfile

| Instruction | Description | Example |
|------------|-------------|---------|
| FROM | Specifies base image | `FROM ubuntu:20.04` |
| RUN | Executes commands during build | `RUN apt-get update && apt-get install -y curl` |
| COPY | Copies files from local machine to image | `COPY app/ /usr/src/app/` |
| ADD | Similar to COPY, can download URLs | `ADD https://example.com/file.tar.gz /app/` |
| WORKDIR | Sets working directory | `WORKDIR /usr/src/app` |
| CMD | Default command when container starts | `CMD ["npm", "start"]` |
| ENTRYPOINT | Configures container as executable | `ENTRYPOINT ["python", "app.py"]` |
| ENV | Sets environment variables | `ENV PORT=8080` |
| EXPOSE | Informs about container's listening port | `EXPOSE 80` |
| VOLUME | Creates mount point for volumes | `VOLUME /data` |
| LABEL | Adds metadata to image | `LABEL maintainer="you@example.com"` |
| ARG | Defines build-time variables | `ARG VERSION=1.0` |
| ONBUILD | Adds triggers for derived images | `ONBUILD RUN apt-get update` |

### Example: Python Flask Application

#### app.py
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Welcome to OPS3!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

#### requirements.txt
```
flask==2.0.1
```

#### Dockerfile
```dockerfile
FROM python:3.10-slim
WORKDIR /usr/src/app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python3", "app.py"]
```

### Building the Docker Image
```bash
$ docker build -t my-python-flask-app .
```

### Running the Container
```bash
$ docker run -p 5000:5000 my-python-flask-app
```

## 8. UPLOADING IMAGES TO A CENTRAL REGISTRY

### Login to Docker Hub
```bash
$ docker login
```

### Tag the Image
```bash
$ docker tag my-python-flask-app:v1 [dockerhub username]/my-python-flask-app:v1
```

### Push to Docker Hub
```bash
$ docker push [dockerhub username]/my-python-flask-app:v1
```

### Understanding Image Layers
Docker uses a union file system where each instruction in a Dockerfile creates a layer. This enables efficient image building and storage.

## 9. DEPLOYING A MULTI-CONTAINER APPLICATION

### Docker Compose

Example docker-compose.yml:
```yaml
services:
  db:
    image: mysql:8.0
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: smart_home
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - smart_home_network

  app:
    build:
      context: ./app
    container_name: flask_app
    depends_on:
      - db
    networks:
      - smart_home_network

  nginx:
    image: nginx:alpine
    container_name: nginx_server
    ports:
      - "80:80"
    networks:
      - smart_home_network

networks:
  smart_home_network:
    driver: bridge

volumes:
  mysql_data:
```

### Running Docker Compose
```bash
$ docker-compose up
```

## Conclusion
Docker provides a powerful platform for containerizing applications, offering consistency, portability, and efficiency across different computing environments.
