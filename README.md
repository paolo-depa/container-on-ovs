# Connecting containers using OVS

## Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
- [Benchmarking](#bench)

## About <a name = "about"></a>

This project is targeted to giving a quick and easy way to connect two (or more) containers using ovs.
The project makes use of podman as container engine, but docker can be used as well with minimal changes.

## Getting Started <a name = "getting_started"></a>

### Building the image

The to-be-used container image is built using the Dockerfile in "images" folder: to build it, enter the directory and run

_podman build -t ovs-container ._

Once done, verify that the image has been added locally:
_podman images | grep ovs-container_

### Running the containers

Once the image is added, open two different console, and run:

- _podman run -it --name bci_0  --net=none localhost/ovs-container:latest_
- _podman run -it --name bci_1  --net=none localhost/ovs-container:latest_

The --net=none switch will make the container start only with loopback interface.

### Connecting containers and ovs

First create a dedicated ovs bridge:

_ovs-vsctl add-br container-br0_

assign it an IP and activate it:

_ip addr add 192.168.10.1/24 dev container-br0_
_ip link set container-br0 up_

Optionally, setup a NAT rule on the bridge interface and allow its traffic to be forwarded:

_iptables -t nat -A POSTROUTING -o container-br0 -j MASQUERADE_
_iptables -A FORWARD -i container-br0 -j ACCEPT_

Now we are going to use the _ovs-docker_ magic script, shipped with ovs: it creates a port in a ovs bridge and injects it in the container.
As I'm using podman instead of docker, I had 2 options:
* Rewrite the script itself to support podman ( feel free to do it [here](https://github.com/openvswitch/ovs/blob/master/utilities/ovs-docker) )
* Cheating a little, creating a symbolic link named "docker" pointing to "podman" executable (_ln -s /usr/bin/podman /usr/bin/docker_)

...I choose the second... and it works! :-D

Let's create the ports:

- ovs-docker add-port container-br0 eth0 bci_0 --gateway=192.168.10.1 --ipaddress="192.168.10.10/24"
- ovs-docker add-port container-br0 eth0 bci_1 --gateway=192.168.10.1 --ipaddress="192.168.10.20/24"

Using the console opened on creation of bci_0 and bci_1 (or attaching a new one - _podman attach bci-[0-1]_), the eth0 interface having the proper IP configured shows up when running:

_ip addr_

## Benchmarking <a name = "bench"></a>

From the bci_0 console run:

_iperf3 -s_

then from the bci_1 console run:

_iperf3 -c 192.168.10.10_

Feel free to play with iperf args: I suggest seeing how performances are modified by using -d, -t and -P NUM, 
