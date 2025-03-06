+++
title = 'Introduction to Docker'
date = '2024-12-10T14:53:04-03:00'
+++

# What is Docker?

### Introduction

- Imagine you're working on a project with a team.
- Let's assume that this project uses a specific version of Python.
- Each team member has to set everything up manually (dependencies, environment variables).
- This brings a variety of problems:
  - "It doesn't work for you, but it works fine on my computer."
  - If we have multiple projects, we need to spend a lot of time configuring them.

### To avoid this We use Containers

A basic and high-level definition of a container is: "A container is a folder or package with everything your application needs to run."

Thanks to a container, we don't have to worry about configuring the project, as everything it needs is inside the container.

The only thing we would need is something to manipulate and manage these containers, and that's where Docker comes in.

## Docker

Docker is/has/can:

- It has a large community, so there are tools and libraries available (dockerhub).
- A tool for managing containers.
- One of the key tools in the DevOps area.
- It simplifies CI/CD processes by providing environments for each stage.
- Containers use fewer resources than virtual machines (explained later).
- Applications run the same way, regardless of the machine they are on (OS agnostic).
- Uses less space because it uses a layer-based file system (explained later).

# How to install Docker?

On Docker's official website, you can find how to install Docker engine on different Linux distributions:

https://docs.docker.com/engine/install/[DISTRO]/

For Windows, you can install Docker Desktop directly (Docker engine with a UI).

Once the installation is complete, to verify everything is working correctly, run this command: `$ docker run hello-world.`

# Show the container already

Before getting to the container, let's talk about the Dockerfile, which creates the image, and from the image, the container is created.

Dockerfile ------> Image --------> Container

The Dockerfile is a text file (yaml) that has the instructions on how the image will be built; it's like creating a mold.

What do I do to the Dockerfile to create the image? Run this command: `$ docker build -t image-name ./dockerfile-directory .`

The image would be the mold, and from the image, a container is created (it can create more than one).

```bash
$ docker run image-name
```

# Anatomy of a Dockerfile

Basically, a Dockerfile describes the necessary steps to create an image, and it is usually placed in the root directory of the app.

Each line in the Dockerfile creates a new layer of an image that Docker will store. Essentially, when creating new images, Docker only needs to create layers of images that differ from the previous one.

```bash
FROM node:0.10 ---> base image

MAINTAINER Anna Doe anna@example.com --> metadata

LABEL "rating"="Five Stars" "class"="First Class" --> metadata

USER root (even though containers provide OS isolation, they still run on the host's kernel; in production containers, always run as a non-privileged user).

-- Set environment variables to simplify the image creation process, keeping the Dockerfile simpler and following the DRY (Don't Repeat Yourself) principle.

ENV APP /data/app ENV SCPATH /etc/supervisor/conf.d

RUN apt-get -y update --> this will make the build slower; if you can find an updated image, use that instead.

#### The daemons (this installs some dependencies I need to use and also forms the file structure I need)

RUN apt-get -y install supervisor
RUN mkdir -p /var/log/supervisor

#### Supervisor Configuration

-- ADD is used to copy files from the local filesystem to your image (once the image is created, I can use it without having access to the original filesystem, as the files were copied). ADD ./supervisord/conf.d/* $SCPATH/

#### Application Code

The order of commands affects execution time. It's optimal to place commands that change between images at the bottom.

ADD .js $AP/ WORKDIR $AP RUN npm install

CMD runs the command that starts the process I want to run inside the container.

CMD ["supervisord", "-n"]
```

### The best practice is to run only one process per container.

## CONTAINERS VS VIRTUAL MACHINES

Yes, containers are nice, but can't I just make a virtual machine, and it's the same?

Why aren't containers and virtual machines the same? Because if they were the same, they would be called the same thing. Goodbye.

A little journey through different eras of software development.
![K8s Virtualization](https://kubernetes.io/images/docs/Container_Evolution.svg)

In the traditional stage, physical servers were used, and we couldn't allocate resources for different parts of the server. In the case of running two applications on the same server, if one takes too many resources, it would crash the other. The solution was to use multiple servers, but the problem with this is the high cost of scaling physical servers and what to do with unused resources.

Virtualized Deployments: Another solution was to run virtual machines on a server. Virtualization allows us to isolate and secure processes since each virtual machine runs its own operating system, preventing others from accessing it. Virtualization optimizes the use of resources on the physical server, and it scales better since we can use a group of physical servers with disposable virtual machines.

Containers: Unlike virtual machines, containers share the operating system between the applications, but they have their own filesystem, CPU, and memory. What makes them very useful is that they are portable across different operating systems and cloud providers.

## How are processes isolated in containers?

Kernel Namespaces, a feature of Linux.
Docker usa:

- PID (process ID): Each container has its own set of process IDs, and inside the container, processes have a different PID from the host, preventing the container from interacting with processes outside of it.

- Network: Isolates network interfaces, IPs, and routing tables. Each container has its own network stack, so it has its own IP and network configuration, separate from the host.

- Mount: This is a way to mount and unmount filesystems without affecting the host.

- UTS: For isolating the hostname and domain name, useful for network configurations.

- User: This allows mapping of user and group IDs inside the container, providing security by not giving direct access to the host's root privileges (very important).

In Docker: Separate namespaces are created for the container, which creates its own isolated environment.

## Who handles the resources used by each container?

Control groups (cgroups), another feature of the Linux kernel, isolate resources within each container (CPU, memory, and disk operations).

In the case of Docker, it places the container's processes in cgroups, which control and limit the container's processes, ensuring good performance.

# If you want to dive deeper into these last topics

https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504

https://medium.com/@saschagrunert/demystifying-containers-part-ii-container-runtimes-e363aa378f25

https://medium.com/@saschagrunert/demystifying-containers-part-iii-container-images-244865de6fef
