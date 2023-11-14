# Project Title

## Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
- [Benchmarking](#bench)

## About <a name = "about"></a>

This project is targeted to giving a quick and easy way to make two or more containers using ovs.
The project makes use of podman as container engine, but docker can be used as well with minimal changes.

## Getting Started <a name = "getting_started"></a>

### Building the image

The to-be-used container image is built using the Dockerfile in "images" folder: to build it, enter the directory and run

_podman build -t ovs-container ._

Once done, verify that the image has been added locally:
_podman images | grep podman build -t ovs-container ._

### Running the containers

Once the image is added, open two different console, and run:

* _podman run -it --name bci_0  --net=none localhost/ovs-container:latest_
* _podman run -it --name bci_1  --net=none localhost/ovs-container:latest_

The --net=none switch will make the container start only with loopback interface.




## Benchmarking <a name = "bench"></a>

Add notes about how to use the system.
