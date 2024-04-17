---
layout: post
title: "Systemd and Linux Containers"
---

Systemd is a system and service manager for Linux operating systems. It is designed to replace the traditional SysVinit init system and provides a range of features for managing system processes, services, and resources. Some key features of systemd include:

- **Service Management**: Systemd manages system services, daemons, and other processes using unit files. It provides utilities like systemctl to start, stop, enable, disable, and manage services

- **Dependency Management**: Systemd allows services to define dependencies on other services and resources. It ensures that services start up in the correct order and that dependencies are satisfied before starting a service

- **Parallel Startup**: Systemd supports parallel startup of services, improving boot times by starting services in parallel rather than sequentially

- **Logging and Journaling**: Systemd includes a logging and journaling system called systemd-journald, which collects and stores system logs in a structured format. It offers features like log rotation, compression, and encryption

- **Resource Control**: Systemd provides mechanisms for controlling and managing system resources such as CPU, memory, and I/O. It allows administrators to set resource limits, control process priorities, and manage resource usage

- **Socket Activation**: Systemd supports socket-based activation, allowing services to be started on-demand when a connection is made to a particular socket. This can improve system efficiency and reduce resource usage. This has a certain similarity with the concept of `serverless`

- **Container Integration**: Systemd includes support for managing containers, integrating with container technologies like Docker and LXC. It provides utilities for starting, stopping, and managing containerized services

Systemd has become the standard init system for many Linux distributions, including Debian, Ubuntu, CentOS, Fedora, and others.

## Managing Docker containers using Systemd

Creating a container with systemd involves setting up a systemd unit file to manage the container process. Here's a basic example of how to create a container using systemd:

### Create a Unit File
Create a systemd unit file (with a `.service` extension) to define the container service. This unit file will specify how the container should be managed by systemd.

```bash
# /etc/systemd/system/nginx.service

[Unit]
Description=My Nginx Container
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker run --rm \
	  --name nginx \
	  --network host \
	  nginx
Restart=always
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

We added few customizations such as container name, and network mode.

### Enable and Start the Service

Once the unit file is created, enable and start the systemd service to launch the container.

```bash
sudo systemctl daemon-reload
sudo systemctl enable nginx.service
sudo systemctl start nginx.service
```

This will start the container using the specified Docker image and manage it as a systemd service.

### Verify Status

We can verify the status of the container systemd service using systemctl.

```bash
sudo systemctl status nginx.service
```

This will display information about the service/container, including its current status.

We can also verify the same logs using the usual docker logs command:

```bash
sudo docker logs nginx
```
That's it! We've now created a container using systemd. We can further customize the unit file to specify additional parameters such as environment variables, volumes, network settings, and more, depending on our requirements.

## Creating containers using Systemd

systemd-nspawn is a tool provided by systemd for creating and managing lightweight containers (similar to chroot environments) on Linux systems. It allows us to run a containerized environment using namespaces, cgroups, and file system isolation, without the need for additional container management software like Docker or LXC.

Systemd-nspawn provides:

- Namespace Isolation: systemd-nspawn utilizes Linux namespaces to provide process, network, and file system isolation for the containerized environment. This allows multiple containers to run on the same host without interfering with each other.

- Minimal Overhead: As a lightweight container solution, systemd-nspawn has minimal overhead compared to full-fledged container runtimes like Docker. It leverages existing Linux kernel features for resource isolation and management.

- File System Bind Mounts: systemd-nspawn supports bind mounts, allowing us to mount directories from the host system into the container environment. This enables data sharing and access to host resources within the container.

- Container Lifecycle Management: systemd-nspawn provides commands for creating, starting, stopping, and managing containers. We can easily create and launch containers using a base image, similar to a chroot environment.

- Networking: systemd-nspawn can configure network interfaces for the containerized environment, enabling network connectivity for containerized applications. It supports various network configurations, including bridged networking and network namespaces.

- Integration with systemd: systemd-nspawn integrates with systemd for process management and supervision. It leverages systemd's unit files and service management capabilities for controlling containerized processes.

It is particularly useful for testing, development, and deployment scenarios where a full container orchestration platform may be unnecessary or too complex. However, it lacks some features found in more specialized container runtimes, such as image management and container orchestration.

To create a container using systemd-nspawn, we'll need a base image or a directory structure that serves as the root file system for the container. Here's a basic outline of the steps to create a container using systemd-nspawn:

### Create a Directory for the Container

First, we create a directory that will serve as the root file system for our container.

```bash
sudo mkdir /srv/my-jammy-container-root
```

### Install a Base System

To create a container with a specific Linux distribution, we can use tools like debootstrap or pacstrap to install a base system into the container directory.

For example, to install a minimal Ubuntu system:

```bash
sudo apt install debootstrap
sudo debootstrap --arch=amd64 jammy /srv/my-jammy-container-root
```

### Enter the Container Environment

We use systemd-nspawn to enter the container environment.

sudo systemd-nspawn --directory=/srv/my-jammy-container-root

This command will start a shell session within the container environment, allowing us to interact with the containerized file system.

### Customize the Container

Once inside the container environment, we can customize the file system, install packages, configure services, etc.

### Exit the Container Environment

Once we're done customizing the container, we can leave the environment using `exit` or `Ctrl+D`.

### Start the Container

We can start the container using systemd-nspawn, specifying the container directory as the root file system.

```bash
sudo systemd-nspawn --directory=/srv/my-jammy-container-root --boot --network-veth
```

This command will start the container with its own network namespace, allowing network communication with the host and other containers.

The END.
