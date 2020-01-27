Docker takes care of the networking aspects so that container can communicate with other containers and also with the Docker Host.

By default if we install docker in to our system it will create "docker0" network interface
Docker networking subsystem is pluggable, using drivers.

There are several drivers available by default, and provides core networking functionality.

* Bridge
* Host
* Overlay
* Macvlan
* none

"docker network ls"
"docker inspect bridge"
docker container run -dt --name hostnetwork --network host ubuntu

apt-get install net-tools
apt-get install iputils-ping
 
 BRIDGE NETWORK
 
 A bridge network uses a software bridge which allows containers connected to the same bridge network to communicate,
 while providing isolation from containers which are not connected to that bridge network
 
 Bridge is the rdefault network driver for Docker.
 If we do not specify a driver, this is the type of network you are creating.
 When you start Docker,a default bridge network (also called bridge) is created.
 automatically, and newly-started containers connected to it unless otherwise specified.
 
 We also can create User-Defined Bridege Network which are superior to the default bridge

USER DEFINED BRIDGE

"brctl show" it will list out the bridge network information

brctl commadn comes with "bridge-utils" package we can install it using yum install bridge-utils

There are various difference between default and user defined bridge.

* User defined bridges provid better isolation and interoperability between containerized applications.
* User defined bridges provide automatic DNS resolution between containers.
* Containers can be attached and detached from user-defined networks on the fly
* Each user-defined network creates a configurable bridge.
* Linked containers on the default bridge network share environment variable

"docker network ls"
"docker network create --driver bridge mybridge
"docker container run -dt --name mybridge01 --network mybridge ubuntu"
"docker container run -dt --name mybridge02 --network mybridge ubuntu"
brctl show

HOST NETWORK

 This driver removes the network isolation between the docker host and the docker containers to use the host's networking directly.
 For instance, if you run a container which binds to port 80 and you use host networking, the container's application will be available on port 80
  on the host's IP address

docker container run -dt --name myhost --network host ubuntu

NONE NETWORK

If you want to completely disable the networkign stack on a container, you can use the none network.
This mode will not configure any IP afrom the container and doesn't have any access to the external network as well as for oher containers.

docker container run -dt --name nonenetwork --network none alpine
ifconfig # no network will be present in the container

PORT PUBLISHING

docker container run -dt --name webserver -p 80:80 nginx
docker container run -dt --name webserver -P nginx  # this will assign random port
