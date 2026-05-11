# Docker and Containers

This repository contains the work and study material for **Module 7**, which focuses on Docker, containers, container images, Docker architecture, and essential Docker commands for development and debugging. The module builds on earlier DevOps topics by showing how applications can be packaged in portable environments and run consistently across systems.

## Overview

In this module, the main focus is understanding how containers simplify application deployment. Instead of installing every dependency directly on a local machine, a container packages the application, configuration, and required dependencies together in one isolated environment. This makes it easier for developers and operations teams to share, test, and deploy applications with fewer environment-related issues.

The notes also explain the role of container repositories, where container images are stored and shared. These repositories can be private for company use or public, such as Docker Hub, where ready-made container images for many applications are available.

## Learning Objectives

By working through this module, the following concepts are covered:

- Understand what a container is and why containers improve software deployment.
- Learn the difference between a Docker image and a running container.
- Explore how Docker packages applications together with dependencies and configuration.
- Understand Docker architecture and the main components of Docker Engine.
- Review common Docker commands used for running, managing, and debugging containers.
- Compare Docker containers with virtual machines in simple terms.
- Learn the basic process of installing Docker Desktop on Windows.

## Why Containers Matter

Before containers, setting up an application for testing or development often required every team member to manually install the required binaries, libraries, and dependencies on their own machine. Because installation steps vary across operating systems, this often caused errors, inconsistencies, and wasted setup time.

Containers solve this by packaging the application and everything it needs into one portable unit. Since the container runs in its own isolated environment, the same package can work more reliably across development, testing, and deployment stages.

## Container vs Image

A **Docker image** is the packaged artifact that contains the application, dependencies, and configuration. It is not actively running; it is simply the reusable package that can be moved between systems.

A **container** is the running instance of that image. Once an image is started on a machine, Docker creates a container environment where the application runs.

In simple terms:

- If it is stored but not running, it is an image.
- If it is running, it is a container.

## Docker vs Virtual Machines

Docker containers and virtual machines both help isolate applications, but they do it differently.

- A **virtual machine** includes a full guest operating system, which makes it larger and heavier.
- A **Docker container** shares the host operating system kernel and only packages the application and its dependencies, which makes it lighter and faster to start.

Because of this, containers are usually more efficient for modern application deployment, while virtual machines are often used when a full separate operating system is needed.

## Docker Desktop Installation

Basic steps to install Docker Desktop on a Windows PC:

1. Go to the [Docker website](https://www.docker.com/products/docker-desktop/).
2. Download Docker Desktop for Windows.
3. Run the installer.
4. Follow the setup instructions shown during installation.
5. Restart the computer if required.
6. Open Docker Desktop and wait for the engine to start.
7. Verify the installation by opening a terminal and running `docker --version`.

## Docker Architecture

When Docker is installed, Docker Engine is installed as well. The notes describe Docker as having three major parts:

- **Docker Server**: Manages images and containers, including pulling, storing, starting, and stopping them.
- **Docker API**: Allows communication with the Docker server programmatically.
- **Docker CLI**: The command-line interface used to send Docker commands to the server.

The Docker server also includes important internal functionalities:

- **Container runtime** for container lifecycle tasks such as starting and stopping containers.
- **Volumes** for persisting data outside the container lifecycle.
- **Networking** for container communication and network setup.
- **Image build functionality** for creating custom Docker images.

## Main Docker Commands

Below are some of the key Docker commands introduced in this module.

| Command | Purpose | Example |
|---------|---------|---------|
| `docker pull` | Downloads an image from a container repository. | `docker pull nginx` |
| `docker run` | Creates and starts a container from an image. | `docker run nginx` |
| `docker run -d` | Runs a container in detached mode, meaning in the background. | `docker run -d nginx` |
| `docker run -p 8080:80` | Maps a host port to a container port. | `docker run -p 8080:80 nginx` |
| `docker run --name web-app` | Starts a container with a custom name. | `docker run --name web-app nginx` |
| `docker start` | Starts an existing stopped container. | `docker start web-app` |
| `docker stop` | Stops a running container. | `docker stop web-app` |
| `docker ps` | Shows currently running containers. | `docker ps` |
| `docker ps -a` | Shows all containers, including stopped ones. | `docker ps -a` |
| `docker images` | Lists downloaded Docker images on the system. | `docker images` |
| `docker exec -it` | Opens an interactive shell or command session inside a running container. | `docker exec -it web-app bash` |
| `docker logs` | Displays logs from a running or stopped container. | `docker logs web-app` |

## Debugging Commands

A few commands are especially useful during troubleshooting:

- `docker logs <container-name>` helps check application output and errors.
- `docker exec -it <container-name> bash` allows interactive access inside the container.
- `docker ps` and `docker ps -a` help confirm container status.

These commands are useful when verifying whether a container started properly, checking why an application failed, or inspecting the environment inside the container.

## Repository Purpose

This repository is intended to document the concepts, commands, and hands-on understanding developed in Module 7. It serves as a study reference for Docker fundamentals and helps connect containerization concepts with the DevOps tools and workflows covered in earlier modules.
