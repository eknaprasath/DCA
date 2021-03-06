Docker containers can have security vulnerabilites
If blindly pulled and if containers are running in production, it can result in breach.

DTR allows us to perform security scan for the containers.
These scan can perform "On Push" or even manually


CONFIGURE CONTAINER SCANNING WITH DTR

Enable the vulnerability scanning features and try with pushing the image to repo

OVERVIEW OF UCP CLIENT BUNDLES

  A client bundle is a group of certificates downloadable directly from the Docker Universal Control Plane (UCP)
  
  Depending on the permission associated with the user, you can now execute docker swarm commands from your remote machine that take effect on the remote cluster
  
For example:

  * You can create a new service in UCP from your laptop.
  * Login to remote container from your laptop without SSH (Via API)
  

UCP Client bundle is associated with user so we can find them from -> my profile
create a new bundle and download the bundle

launch a container

docker container run -it -dt --name myubuntu ubuntu bash

docker ps
docker cp "ucp-bundle-admin.zip" myubuntu:/tmp

docker container exec -it myubuntu bash
cd /
mkdir /ucp
cp /tmp/ucp-bundle-admin.zip /ucp
apt-get update
apt-get install unzip

unzip ucp-bundle-admin
nano env.sh


eval "$(<env.sh)"

to verify 
echo $DOCKER_HOST

curl -sSL https://get.docker.com/ | sh

docker service create --name mydemoservice nginx

goto ucp gui and check for the created service


INTEGRATING LDAP WITH UCP

Goto admin setting -->Authentication & Authorization --> LDAP ENABLE 

LINUX NAMESPACES

Docker uses a technology called namespaces to provide the isolated work space called the container
The namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.

Namespace A                         Namespace B
UTS PID                             UTS   PID

docker container run -dt --name mybusybox busybox sh
docker ps
ps -ef

docker container exec -it mybusybox bash

ps -ef 

unshare -fp --mount-proc /bin/bash

unshare --uts /bin/bash
hostname
hostname mynamespace

Currently Linux provides six different types of namespaces as follows:

* Inter-Process Communication (IPC)
* Process(PID)
* Network(net)
* User Id (user)
* Mount (mnt)
* UTS

CONTROL GROUPS (CGROUPS)
Control groups (cgroups) is a Linux kernel feature that limits, accounts for, and isolates the resource usage(CPU,memory,disk I/O, network, etc.)
of a collection of processes.

Control grups is a Linux kernal feature but docker support primaryly CPU and Memory

LIMITING MEMORY

docker run --help
-m or --memory in bytes

docker container run -dt --name mymemory01 -m 256m busybox
docker container exec -it mymemory01 sh

free -m

cat /sys/fs/cgroup/memory/memory.limit_in_bytes


LIMITING CPU
docker run --help

There are various ways in which you can limit the CPU for containers, these include:

Approaches                        `Description
--cpus=<value>            If you has 2 CPUs and if you set --cpus=1 then container is guranteed at most one CPU. you can even specify --cpu=0.5

--cpuset-cpus             Limit the specific CPUs of cores a container can use. A comma-separated list or hyphen-separated range of CPUs a container can use
                          1st CPU is numbered 0
                          2nd CPU is numbered 1
                          
                          Value of 0-3 means usage of first, second, third and fourch CPU.
                          Value of 1,3 means second and fourth CPU.



docker container run -dt --name mycpu01 --cpus=1.5 busybox sh

docker container run -dt --name cpu3 --cpuset-cpus=0,1 busybox sh 


RESERVATIN VS LIMIT

  By default, a container has no resource constraints and can use much of a given resource as the host's kernel scheduler allows.
  
  It is important not to allow a running container to consume too much of the host machine's memory.
  
  One Linux hosts, if the kernel detects that there is not enough memory to perform important system functions, it throws an "Out Of Memory" exception,
  and starts killing processes to free up memory.
  
  
  Limit imposes an upper limit to the amount of memory that can potentially be used by a Docker container.
  
  Reservations, on the other hand, is less rigid.
  
  When the system is running low on memory and there is contention, reservation tries to bring the container's memory consumption at or
  below the reservation limit.
  
  LIMIT is hard limit
  RESERVATION is soft limit
  
  
  docker container run -dt --name container01 --memory-reservation 250m busybox sh #soft limit
  
  docker container run -dt --name container02 -m 500m --memory-reservation 250m busybox sh #hard limit with softlimit reservation
  
SWARM MTLS ( Mutual Transport Layer Security )

  The nodes in a swarm use mutual Transport Layer Security(TLS) to authenticate, authorize, and encrypt the communications with other nodes in the swarm.
  
  By default, the manager node generates a new root Certificate Authority (CA) along with a key pair, which are used to secure communications with other nodes that join the swarm.
  
  You can specify your own externally-generated root CA, using the --external-ca flag of the docker swarm init command.
  
  Each time a new node joins the swar, the manager issues a certificate to the node.



docker swarm init

cd /var/lib/docker/swarm/certificates

  The manager node also generates two tokens to use when you join additional nodes to the swarm: one "worker" token and one "manager" token
  
  Each token includes the digest of the root CA's certificate and a randomly generated secret.
  
  When a node joins the swarm, the joining node uses the digest to validate the root CA certificate from the remote manager.
  
  
ROTATING THE CA CERTIFICATE
   Run "docker swarm ca --rotate" to generate a new CA certificate and key.
   
   In the above command, docker generated a cross-signed certificate signed by previous old CA.
   
   After every node in the swarm has a new TLS certificate signed by the new CA. Docker forgets about te old CA certificate
   and key material, and tells all the nodes to trust the new CA certificate only.
   
   Join tokens also changes accordingly.
   
docker swarm join-token worker

copy the token and run it in a new node

go to worker node following location
cd /var/lib/docker/swarm/certificates
ls

docker swarm leave
ls 
all certificates will be removed

go to master node
docker node rm worker01

docker swarm ca --rotate

Go to the same worker node again and run the previous join token 

Generate new join token and execute in worker

------------------------------
If we rotate the certificate to the exising cluster nothing will happen for cluster envirionment but keys will be changed in all nodes

go to worker node and do the 

md5sum swarm-node.crt
md5sum swarm-node.key

go to master and rotate the key and verify again

  By default, each node in the swarm renews its certificate every three months.
  
  You can configure this interval by running the "docker swarm update --cert-expiry <TIME PERIOD>
  
  In the event that a cluster CA key or a manager node is compromised, you can rotate the swarm node CA so that none of the nodes trust 
  certificates signed by the old root CA anymore.


MANAGING SECRETS IN DOCKER SWARM

  Docker secrets to certrally manager this data and securely transmit it to only those containers that need access to it.
  
  Secrets are encrypted during transit and at rest in a Docker swarm.
  
  A given secret is ony accessible to those services which have been granted explicit access to it, and only while those service tasks are running.

docker swarm init

docker swarm init --advertise-addr <IP>

docker secret ls

vi db_credentials.txt

username: dbadmin
pass: dbadmin@123

docker secret create dbcredentials db_credentials.txt

docker secret ls

you will not get the credentials info from the inspect command.

docker secret inspect dbcredentials


docker service create --name webserver --secret dbcredentials nginx

docker ps

docker container exec -it <container name or id> bash

cd /run
ls

cat dbcredentials

you can able to view the secrets now

DOCKER CONTENT TRUST

When we are downloading image from network or from internet, integrity becomes a true concern.

Content trust gives you the ability to verify both the integrity and the publisher of all the data received from a registry over any channel.

This can be achieved with the help of digital signatures.



Before enable the docker content trust pull a image DTR


docker pull example.com/admin/scanserver:ubuntu

export DOCKER_CONTENT_TRUST=1

docker rmi example.com/admin/scanserver:ubuntu
docker pull example.com/admin/scanserver:ubuntu

will get output like " No valid trust data for ubuntu"

docker pull nginx

docker trust inspect nginx


OVERVIEW OF DOCKER GROUP

useradd ekna
su - ekna

docker ps #will get permission denied 

add user to "docker" group and that user can do the docker related commands

usermod -aG docker dockeruser



LINUX CAPABILITIES FOR DOCKER

  There are several types of capabilities which Linux provides to have granular acces at the application level.
  
  
  CAPABILITY          ACTION
  
  SETPCAP             Yes
  CHOWN               Yes
  SYS_MODULE          No
  LINUX_IMMUTABLE     No
  
  docker run -dt --name cap-demo amazonlinux
  docker exec cap-demo bash
  whoami
  touch test.txt
  chattr
  yum whatprovides chattr
  yum install e2fsprogs
  chattr -i test.txt
  
  go to docker documentations and check for capability key 
  
  docs.docker.com/engine/reference/run
  
  scroll down for capability
  
  docker run -dt --name cap02 --cap-add LINUX_IMMUTABLE amazonlinux
  docker exec --name cap02 bash
  yum install e2fsprogs 
  touch test.txt
  chattr +i test.txt
  
  rm test.txt
  echo "Hi" > test.txt
  
  chattr -i test.txt
  


PRIVILEDGED CONTAINERS

  By default, docker container does not have many capabilities assigned to it.
  
    * Docker containers are also not allowed to access any devices.
    
  Hence, by default, docker container cannot perfor various use-cases like run docker conatiner inside a docker container.
  
  PRIVILEGED CONTAINERS
  
    Privileged container can access all the devices on the host as well as have configuration in AppArmor or SELinux to allow the container
    nearly all the same access to the host as processes running outside containers on the host.
    
      * Limits that you set for privileged containers will not be followed.
      * Running a privileged flag gives all the capabilities to the container.
      
      
      docker run -dt --name amazonlinux amazonlinux bash
      docker ps
      on host machine
      cd /dev
      ls
      docker exec -it amazonlinux bash
      cd /dev/
      ls
      
      Now run a privileged container and see
      
      docker run -dt --privileged amazonlinux bash
      docker ps
      docker exec -it containerid bash
      
      cd /dev/
      ls
      
      
  
  
  
  
