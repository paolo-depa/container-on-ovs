# Connecting containers using OVS

## Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
- [Benchmarking](#bench)

## About <a name = "about"></a>

This project aims to give a quick and easy way to connect two (or more) containers using openvswitch (aka ovs).
The project makes use of podman as container engine, but docker can be used as well with minimal changes.

## Getting Started <a name = "getting_started"></a>

### Building the image

The to-be-used container image is built using this [Dockerfile](images/Dockerfile): to build it, enter the directory and run

> podman build -t ovs-container .

Once done, verify that the image has been added locally:
> podman images | grep ovs-container

### Running the containers

Once the image is added, open two different console, and run:

1. > podman run -it --name bci_0  --net=none localhost/ovs-container:latest
1. > podman run -it --name bci_1  --net=none localhost/ovs-container:latest

The --net=none switch will make the container start only with loopback interface.

### Connecting containers and ovs

First create a dedicated ovs bridge:

> ovs-vsctl add-br container-br0

assign it an IP and activate it:

> ip addr add 192.168.10.1/24 dev container-br0
> ip link set container-br0 up

Optionally, setup a NAT rule on the bridge interface and allow its traffic to be forwarded:

> iptables -t nat -A POSTROUTING -o container-br0 -j MASQUERADE;

> iptables -A FORWARD -i container-br0 -j ACCEPT

Now we are going to use the *ovs-docker* magic script, shipped with ovs: it creates a port in a ovs bridge and injects it in the container.
As I'm using podman instead of docker, I had 2 options:
* Rewrite the script itself to support podman ( feel free to do it [here](https://github.com/openvswitch/ovs/blob/master/utilities/ovs-docker) )
* Cheating a little, creating a symbolic link named "docker" pointing to "podman" executable (_ln -s /usr/bin/podman /usr/bin/docker_)

...I choose the second... and it works! :-D

Let's create the ports:

1. > ovs-docker add-port container-br0 eth0 bci_0 --gateway=192.168.10.1 --ipaddress="192.168.10.10/24"
1. > ovs-docker add-port container-br0 eth0 bci_1 --gateway=192.168.10.1 --ipaddress="192.168.10.20/24"

Using the console opened when creating bci_0 and bci_1 (or attaching a new one - _podman attach bci-[0-1]_), the eth0 interface having the proper IP configured shows up when running:

> ip addr

## Benchmarking <a name = "bench"></a>

In the *bci_0* console run:

> iperf3 -s 

then in the *bci_1* console run:

> iperf3 -c 192.168.10.10

Feel free to play with iperf args: I suggest seeing how performances are modified by using -d, -t and -P NUM, 
