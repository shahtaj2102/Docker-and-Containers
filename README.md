# Docker and Containers

This repository documents the hands-on work and study notes for **Module 07 – Containers with Docker**. The module covers everything from understanding what containers are, to running a full local development environment using Docker, building custom images with a Dockerfile, pushing images to a private Nexus Docker registry, deploying applications with Docker Compose, persisting data using Docker Volumes, and migrating Nexus itself to run as a container.

The module builds directly on the DigitalOcean and Nexus workflows from Modules 5 and 6.

## Objectives

By the end of this module, you should be able to:

- Explain what a container is and how it differs from a virtual machine.
- Understand the difference between a Docker image and a running container.
- Use Docker CLI commands to pull, run, manage, and debug containers.
- Set up a local development environment using MongoDB and Mongo Express containers.
- Create and use Docker Networks so containers can communicate.
- Write a Dockerfile to package a Node.js application into a custom image.
- Build, tag, and push a Docker image to a private Nexus Docker registry.
- Write and use a Docker Compose file to manage a multi-container setup.
- Use Docker Volumes to persist data across container restarts.
- Deploy Nexus as a Docker container on a DigitalOcean Droplet.
- Apply Docker best practices for security and image optimization.

## Prerequisites

Before starting, make sure you have:

- Docker Desktop installed (Windows/Mac) or Docker Engine installed (Linux).
- A DigitalOcean account with a Droplet available.
- Nexus Repository Manager running (from Module 6) or set up as a container.
- Basic Linux command-line knowledge.
- Node.js installed locally for running the demo application.
- SSH access to the server.

## What is a Container?

A container is a way to package an application together with all its dependencies and configuration into one portable, isolated unit. That package can be shared between development and operations teams and run consistently across different environments without needing to install anything directly on the host operating system.

Containers are stored in container repositories. Companies often have private repositories for internal images, while Docker Hub serves as the public repository where ready-made images for common applications can be found and pulled.

### Container vs Image

- A **Docker image** is the stored, non-running artifact that packages the application, its dependencies, and configuration. It is the movable unit.
- A **container** is the running instance of an image. When an image is started on a machine, Docker creates the container environment and the application begins executing.

In simple terms: if it is stored but not running, it is an image. If it is actively running, it is a container.

## Docker vs Virtual Machines

Both Docker and virtual machines are virtualization tools, but they work differently.

- A **virtual machine** includes a full guest operating system on top of the host, making it heavier and slower to start.
- A **Docker container** shares the host OS kernel and only packages the application layer. It is lighter, faster to start, and more resource-efficient.

Containers are preferred for modern application deployment when a full isolated OS is not required.

## Docker Architecture

Installing Docker installs the **Docker Engine**, which has three parts:

- **Docker Server**: Responsible for pulling images, storing them, and managing the full container lifecycle (start, stop, remove).
- **Docker API**: The REST API that allows programs to communicate with the Docker server.
- **Docker CLI**: The command-line client used to send commands to the server.

The Docker server itself contains:

- **Container runtime** – handles the container lifecycle.
- **Volumes** – handles persistent data storage for containers.
- **Networking** – configures how containers communicate with each other and the outside world.
- **Image build functionality** – allows building custom images from Dockerfiles.

## Docker Desktop Installation (Windows)

1. Go to the [Docker website](https://www.docker.com/products/docker-desktop/).
2. Download Docker Desktop for Windows.
3. Run the installer and follow the on-screen instructions.
4. Restart the computer if prompted.
5. Open Docker Desktop and wait for the engine to start.
6. Verify installation by running `docker --version` in a terminal.

## Main Docker Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker pull` | Downloads an image from a repository. | `docker pull nginx` |
| `docker run` | Creates and starts a container from an image. | `docker run nginx` |
| `docker run -d` | Runs a container in detached (background) mode. | `docker run -d nginx` |
| `docker run -p host:container` | Maps a host port to a container port. | `docker run -p 8080:80 nginx` |
| `docker run --name` | Starts a container with a custom name. | `docker run --name web-app nginx` |
| `docker start` | Starts an existing stopped container. | `docker start web-app` |
| `docker stop` | Stops a running container. | `docker stop web-app` |
| `docker ps` | Lists currently running containers. | `docker ps` |
| `docker ps -a` | Lists all containers including stopped ones. | `docker ps -a` |
| `docker images` | Lists all downloaded images on the system. | `docker images` |
| `docker logs` | Shows output logs from a container. | `docker logs web-app` |
| `docker logs -f` | Streams container logs live in the terminal. | `docker logs web-app -f` |
| `docker exec -it` | Opens an interactive shell inside a running container. | `docker exec -it web-app bash` |

## Developing with Docker

This section walks through a practical demo using a simple JavaScript and Node.js application connected to a MongoDB database and a Mongo Express UI, all running as Docker containers.

### 1. Clone and Run the Application Locally

```bash
git clone <repo-url>
cd app
npm install
node server.js
```

Access the app at `http://localhost:3000`. Without a database connected, any changes will be lost on refresh.

### 2. Create a Docker Network

Containers in the same Docker network can communicate using just the container name. Create a dedicated network for MongoDB and Mongo Express:

```bash
docker network create mongo-network
```

Verify it was created:

```bash
docker network ls
```

### 3. Run the MongoDB Container

```bash
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongodb \
  --net mongo-network \
  mongo
```

- `-d` runs the container in the background.
- `-p 27017:27017` maps the default MongoDB port.
- `-e` passes environment variables to set root credentials.
- `--name` gives the container a name so other containers can reference it.
- `--net` places the container in the network we created.

### 4. Run the Mongo Express Container

```bash
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=hero123 \
  -e ME_CONFIG_BASICAUTH_USERNAME=user \
  -e ME_CONFIG_BASICAUTH_PASSWORD=hero123 \
  --net mongo-network \
  --name mongo-express \
  -e ME_CONFIG_MONGODB_SERVER=mongodb \
  -e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 \
  mongo-express
```

The `ME_CONFIG_MONGODB_URL` variable is important. Without it, Mongo Express may fail to locate the MongoDB server.

### 5. Open Mongo Express in the Browser

```
http://localhost:8081
```

Log in using the basic auth credentials set above (`user` / `hero123`).

### 6. Create the Database and Collection

In the Mongo Express UI, create a database called `user-account` and inside it a collection called `users`.

### 7. Connect the Node.js App to MongoDB

Update `server.js` to use the MongoDB connection string pointing to `localhost:27017` with the admin credentials set when the container was started.

### 8. Access and Test the Application

```
http://localhost:3000
```

Edit the profile details and save. Refresh the page and the data will persist because it is now written to the MongoDB container. You can confirm the saved data in the Mongo Express UI under the `users` collection.

To check logs at any point:

```bash
docker logs mongodb
docker logs mongo-express
docker logs mongodb -f    # stream logs live
```

## Docker Compose

Instead of running long `docker run` commands for each container, Docker Compose lets you define all containers and their configuration in one structured YAML file.

Docker Compose also automatically creates a shared network for all services defined in the file, so a separate `docker network create` command is not needed.

### Example Docker Compose File

```yaml
version: '3'
services:
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
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_BASICAUTH_USERNAME=user
      - ME_CONFIG_BASICAUTH_PASSWORD=pass
      - ME_CONFIG_MONGODB_SERVER=mongodb
      - ME_CONFIG_MONGODB_URL=mongodb://admin:password@mongodb:27017
    depends_on:
      - "mongodb"

volumes:
  mongo-data:
    driver: local
```

- `restart: always` ensures the container automatically restarts if it crashes or the server reboots.
- `depends_on` tells Docker Compose to start `mongodb` before `mongo-express`.
- The `volumes` section at the bottom (same level as `services`) declares the named volumes used by the containers.

### Starting and Stopping with Docker Compose

Save the file as a `.yaml` file. Indentation is critical in YAML.

Start all containers:

```bash
docker compose -f mongo.yaml up
```

Stop and remove all containers:

```bash
docker compose -f mongo.yaml down
```

> **Note:** Recreating a container will lose all stored data unless Docker Volumes are configured.

## Dockerfile

A Dockerfile is a blueprint for building a custom Docker image from your application. It always has to be named `Dockerfile` with a capital D and no file extension.

### Dockerfile Instructions

| Instruction | Purpose |
|-------------|----------|
| `FROM` | Sets the base image. Every Dockerfile starts with this. |
| `ENV` | Sets environment variables inside the image. |
| `RUN` | Executes Linux commands inside the container during the build. |
| `COPY` | Copies files from the host into the container image. Unlike `RUN cp`, this runs on the host. |
| `WORKDIR` | Sets the default working directory for all following instructions. |
| `CMD` | The entry point command that starts the application when the container runs. Only one `CMD` is used per Dockerfile. |

**Difference between `RUN` and `CMD`:** You can have many `RUN` commands for setup tasks (installing packages, creating directories), but `CMD` is the single entry point that starts the actual application.

### Example Dockerfile for the Node.js App

```dockerfile
FROM node:20-alpine

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PASSWORD=password

RUN mkdir -p /home/app

COPY . /home/app

WORKDIR /home/app

RUN npm install

CMD ["node", "server.js"]
```

- `node:20-alpine` is used as the base image because it already has Node.js installed and Alpine keeps the image small.
- `COPY . /home/app` copies the project files from the host into the container.
- `WORKDIR /home/app` means all following commands execute from inside that directory.
- `npm install` runs inside the container during build to install Node dependencies.
- `CMD ["node", "server.js"]` starts the server when the container launches.

> **Note:** It is better practice to include `npm install` in the Dockerfile rather than running it beforehand on the host. This ensures dependencies are installed in the container environment, not the host environment, and keeps the image self-contained.

> **Note:** Whenever the Dockerfile is modified, the image must be rebuilt.

### Build the Docker Image

Run the following command in the same directory as the Dockerfile:

```bash
docker build -t my-app:1.0 .
```

- `-t my-app:1.0` gives the image a name and version tag.
- `.` tells Docker to look for the Dockerfile in the current directory.

Verify the image was created:

```bash
docker images
```

## Docker Registry – Pushing to Nexus

Once the application image is built, it can be pushed to a private Docker registry hosted on Nexus. This allows other team members or servers to pull the latest image.

### 1. Create a Docker Hosted Repository in Nexus

- Open the Nexus Repository Manager UI.
- Create a new **Docker hosted** repository.
- Under the HTTP connector field, set a dedicated port (e.g., `8083`) so that the Docker client can connect to it directly using IP and port rather than a path.

### 2. Open Firewall Port for Docker Repository

In the DigitalOcean firewall settings, open the port configured for the Docker repository (e.g., `8083`) so it is accessible from the outside.

### 3. Configure Docker Realm in Nexus

Enable the **Docker Bearer Token Realm** in Nexus under Security > Realms. This allows Nexus to issue authentication tokens when a client runs `docker login`.

### 4. Allow Insecure Registry on the Docker Client

Since Nexus is running on HTTP rather than HTTPS, Docker must be told to allow this as an exception. Edit or create the `daemon.json` file:

```bash
# File location: /etc/docker/daemon.json
{
  "insecure-registries" : ["<droplet-ip>:8083"]
}
```

Restart Docker after saving the file.

### 5. Log In to the Nexus Docker Registry

```bash
docker login <droplet-ip>:8083
```

This stores an authentication token in `~/.docker/config.json` on the local machine. This token is reused for all future `push` and `pull` operations against that registry.

### 6. Tag the Image

Before pushing, retag the image to include the registry address:

```bash
docker tag my-app:1.0 <droplet-ip>:8083/my-app:1.0
```

### 7. Push the Image

```bash
docker push <droplet-ip>:8083/my-app:1.0
```

Verify the image appears in the Nexus Docker hosted repository through the UI or the Nexus REST API:

```bash
curl -u <user>:<password> -X GET 'http://<droplet-ip>:8081/service/rest/v1/components?repository=docker-hosted'
```

## Deploying the Application with Docker Compose

Once the custom image is in the registry, it can be added to the Docker Compose file alongside MongoDB and Mongo Express to start the full application on any server.

Add the `my-app` service to the compose file:

```yaml
services:
  my-app:
    image: <droplet-ip>:8083/my-app:1.0
    ports:
      - 3000:3000
```

Because the app is now running as a container (not directly on the host), update `server.js` to connect to MongoDB using the container name instead of `localhost`:

```js
// Change from:
const mongoUrlLocal = 'mongodb://admin:password@localhost:27017';

// To:
const mongoUrlDockerCompose = 'mongodb://admin:password@mongodb:27017';
```

After updating the code, rebuild and re-push the image, then run the compose file on the target server:

```bash
docker login <droplet-ip>:8083    # required for pulling private images
docker compose -f mongo.yaml up
```

Access the full application at `http://localhost:3000` and Mongo Express at `http://localhost:8081`.

## Docker Volumes

By default, all data stored in a container is lost when the container is removed or recreated. Docker Volumes solve this by mounting a storage location from the host file system into the container.

### Volume Types

1. **Host Volumes** – You define both the host path and the container path.

```bash
docker run -v /home/data:/data/db mongo
```

2. **Anonymous Volumes** – Only the container path is defined. Docker picks the host path automatically.

```bash
docker run -v /data/db mongo
```

3. **Named Volumes** – An upgrade to anonymous volumes where you give the host-side volume a name. These are the most practical and commonly used type.

```bash
docker run -v mongo-data:/data/db mongo
```

### Docker Volume Locations on the Host

| OS | Path |
|----|------|
| Windows | `C:\ProgramData\docker\volumes` |
| Linux / macOS | `/var/lib/docker/volumes` |

### Volumes in Docker Compose

Volumes are added to each service under the `volumes` key, and all named volumes are also declared at the top-level `volumes` block:

```yaml
services:
  mongodb:
    image: mongo
    volumes:
      - mongo-data:/data/db    # /data/db is where MongoDB stores its data

volumes:
  mongo-data:
    driver: local
```

The same volume can be mounted to multiple containers if they need to share data.

### Useful Volume Commands

```bash
docker volume ls                        # list all volumes
docker volume inspect <volume-name>     # view details about a volume
```

## Moving Nexus to a Container

Instead of installing Nexus directly on a VM as done in Module 6, Nexus can also be run as a Docker container. This requires only Docker installed on the server, not Java.

### 1. Provision a New Droplet

Create a new DigitalOcean Droplet, configure SSH and firewall access on port 22, and SSH in.

### 2. Install Docker

```bash
sudo apt update
sudo snap install docker
```

### 3. Create a Volume and Run the Nexus Container

```bash
docker volume create --name nexus-data
docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
```

- `-d` runs Nexus in the background.
- `-p 8081:8081` maps the Nexus port.
- `-v nexus-data:/nexus-data` mounts the named volume to persist Nexus data across restarts.
- `sonatype/nexus3` is the official Nexus image from Docker Hub.

Verify Nexus is running:

```bash
docker ps
netstat -lntp
```

Access Nexus at `http://<droplet-ip>:8081`.

> **Note:** The official Nexus image already creates a dedicated `nexus` user inside the container and runs the service under that user, following the security best practice from Module 6 without any manual setup.

## Docker Best Practices

1. **Use official and verified images** whenever available to reduce security risk.
2. **Pin image versions** – always specify a version tag rather than relying on `latest`. More specific is better (e.g., `node:20-alpine` instead of `node`).
3. **Choose leaner base images** – prefer Alpine-based images when no special system libraries are needed. Smaller images are faster to pull, build, and deploy, and have a smaller attack surface.
4. **Order Dockerfile instructions from least to most frequently changed** to take advantage of Docker's build cache and speed up rebuilds.
5. **Use a `.dockerignore` file** to exclude files and directories (like `node_modules`) from being copied into the image during the build step.
6. **Use multi-stage builds** to separate the build environment from the runtime environment, keeping the final image lean by excluding build tools.
7. **Do not run containers as root** – create a dedicated least-privilege user inside the Dockerfile to reduce security risk.
8. **Scan images for vulnerabilities** after building using:

```bash
docker scout cves <image-name>
```

You must be logged in to Docker Hub to use this command.

## Key Commands Reference

```bash
# Pull and run
docker pull mongo
docker run -d -p 27017:27017 --name mongodb --net mongo-network mongo

# Networking
docker network create mongo-network
docker network ls

# Build an image
docker build -t my-app:1.0 .

# Tag and push to private registry
docker tag my-app:1.0 <registry-ip>:8083/my-app:1.0
docker push <registry-ip>:8083/my-app:1.0

# Docker Compose
docker compose -f mongo.yaml up
docker compose -f mongo.yaml down

# Volumes
docker volume create --name nexus-data
docker volume ls
docker volume inspect nexus-data

# Debugging
docker logs <container-name>
docker logs <container-name> -f
docker exec -it <container-name> bash
docker ps
docker ps -a
```

## Notes

- Replace placeholder values like `<droplet-ip>`, `<registry-ip>`, `<user>`, and `<password>` with your actual deployment details.
- Whenever a Dockerfile is updated, the image must be rebuilt and re-pushed to the registry.
- Recreating a Docker Compose environment without volumes will result in data loss.
- Port `8081` is the default Nexus port; port `8083` is used in these notes as the dedicated Docker registry port.
- Port `22` must always remain open on the Droplet for SSH access.
