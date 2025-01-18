
# Docker Basics Guide

## Introduction to Docker
Docker is a platform for developing, shipping, and running applications in isolated environments called containers. Containers bundle an application and its dependencies, ensuring it runs consistently across various environments.

### Key Benefits of Docker:
- Consistency across development and production.
- Lightweight and fast compared to traditional virtual machines.
- Simplifies dependency management.

---

## Installing Docker
1. Update your system's package index:
   ```bash
   sudo apt update
   ```
2. Install Docker:
   ```bash
   sudo apt install docker.io -y
   ```
3. Verify Docker installation:
   ```bash
   docker --version
   ```
4. Add your user to the Docker group (optional for non-root access):
   ```bash
   sudo usermod -aG docker $USER
   ```

---

## Understanding a Dockerfile
A `Dockerfile` defines the instructions to build a Docker image.

### Example: Dockerfile for a Node.js App
```Dockerfile
# Base image
FROM node:14

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy application files
COPY . .

# Expose the application port
EXPOSE 3000

# Start the application
CMD ["node", "server.js"]
```

### Explanation:
- `FROM`: Specifies the base image.
- `WORKDIR`: Sets the working directory in the container.
- `COPY`: Copies files from the host to the container.
- `RUN`: Executes commands (e.g., installing dependencies).
- `EXPOSE`: Exposes a port to allow external access.
- `CMD`: Defines the default command to run when the container starts.

### Commands to Build and Run:
1. Build the image:
   ```bash
   docker build -t my-node-app .
   ```
2. Run the container:
   ```bash
   docker run -p 3000:3000 my-node-app
   ```

---

## Docker Compose Basics
Docker Compose is a tool to manage multi-container applications using a single `docker-compose.yml` file.

### Example: docker-compose.yml
```yaml
version: '3.9'

services:
  app:
    build:
      context: .
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### Commands:
1. Start the services:
   ```bash
   docker-compose up
   ```
2. Stop the services:
   ```bash
   docker-compose down
   ```

### Explanation:
- `services`: Defines containers in the application.
- `build`: Specifies the build context (Dockerfile location).
- `ports`: Maps host ports to container ports.
- `volumes`: Mounts host directories into the container.
- `networks`: Defines custom networks.

---

## Docker Networks
Docker networks allow containers to communicate.

### Commands:
1. List networks:
   ```bash
   docker network ls
   ```
2. Create a network:
   ```bash
   docker network create my-network
   ```
3. Connect a container to a network:
   ```bash
   docker network connect my-network <container_name>
   ```

---

## Docker Volumes
Volumes persist data between container restarts.

### Commands:
1. Create a volume:
   ```bash
   docker volume create my-volume
   ```
2. Use a volume in a container:
   ```bash
   docker run -v my-volume:/data my-node-app
   ```
3. List volumes:
   ```bash
   docker volume ls
   ```

### Example: Using Volumes in docker-compose.yml
```yaml
services:
  app:
    image: my-node-app
    volumes:
      - my-volume:/usr/src/app/data

volumes:
  my-volume:
```

---

## Additional Commands
- View running containers:
  ```bash
  docker ps
  ```
- Stop a container:
  ```bash
  docker stop <container_id>
  ```
- Remove a container:
  ```bash
  docker rm <container_id>
  ```
- Remove an image:
  ```bash
  docker rmi <image_id>
  ```

---

This guide introduces the basics of Docker, setting up a Node.js application, and managing it using Docker Compose, networks, and volumes.
