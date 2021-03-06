Container orchestration is all about managing the life cycle of containers, especially in large, dynamic environemnts,

Container monitorint
maintaining the number of docker containers
Host failure handling

Container orchestration can be used to perform lot of tasks, some of them includes:

 * Provisioning and deployment of containers
 * Scaling up or removing containers to spread application load evenly
 * Movement of containers from one host to another if there is a shortage of resources
 * Load balancing of service discovery between containers
 * Health monitoring of containers and hosts
 
 Container Orchestration Solutions
 
 There are many containers orchestration solutions which are available, some of them popular o
 
 * Docker swarm
 * Kubernetes
 * Apache Mesos
 * Elastic Container Service (AWS ECS)
 
 Docker swarm is a container orchestration tool which is native supported by Docker
 
 create 3 Node cluster
 
 #!/bin/bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl start docker
systemctl enable docker


Initializing Docker Swarm

A node is an instane of the Docker engine participating in the swarm .
To deploy your application to a swarm, you submit a service definition to a manager node
The manager node dispatches units of work called tasks to worker nodes.



Manager Node command:
 docker swarm init --advertise-addr <MANAGER-IP>
 
Worker Nodes command:
 docker swarm join --token worker-token
 
 docker node ls
 
 SERVICES, TASKS AND CONTAINERS
 
 A service is the definition of the tasks to execute on the manager or worker nodes.
 
 docker service create --name webserver --replicas 1 nginx
 
 docker service ls
 
 docker service ps webserver
 
 docker container stop webserver
 
 docker serivce ps webserver
 
 systemctl stop docker
 
 docker service rm webserver
 
 SCALING THE SWARM SERVICE
 
 Once you have deployed a service to a swarm, you are ready to use the docker CLI to 
 scale the number of containers in the service.
 
 Containers running in a service are called "tasks"
 
 docker service scale webserver=5
 
 docker service ps webserver
 
 stop docker in any one of the node and check the functionality
 
 docker service scale webserver=2
 
 docker service create --name service01 --replicas 1 nginx
 
 docker service create --name service02 --replicas 1 nginx
 
 docker service ls
 
 docker service update --replicas 5 service01 # at a time we can scale only one service by using this command
 docker service scale service01=3 service02=3 # using this command we can scale multiple services at a time
 
 
 REPLICATED VS GLOBAL SERVICE
 
 There are two types of services deployments, replicated and global
 For a replicated service, you specify the number of identical tasks you want to run. for example
 you decide to deploy an NGINX service with two replicas, each servicing the same conten.
 
 Global service is a service that runs one task on every node.
 
 Each time you add a node to the swarm, the orchestrator creates a task and the scheduler assigns the task to the new node.
 
 docker service create --name antivirus --mode global -dt ubunt
 
 NODE DRAINING
 
 docker node ls
 
 docker service create --name mywebserver --replicas 5 nginx
 
 docker service ps
 
 docker node update --availability drain swarm03
 docker node ls
 
 docker node update --availability active swarm03
 docker node ls
 
 DOCKER INSPECT
 
 Docker inspect provides detailed information of the Doceker Object
 
 Some of these Docker Object includes:
 
 - Docker Containers
 - Docker Network
 - Docker Volues
 - Docker Swarm Services
 - Docker Swarm Nodes
 
 docker service inspect demotroubleshoot
 docker service inspect demotroubleshoot --pretty
 
 docker node inspect node-name
 docker node inspect node-name --pretty
 
 ADD NETWORKING AND PUBLISHING PORT 
 
 docker service create --name web --replicas 2 nginx
 docker service ls
 docker service ps
 
 docker service create --name webserver --replicas 2 --publish 80:80 nginx
 docker service ls
 
 
 DOCKER COMPOSE OVERVIEW
 
 
 Compose is a tool for defining and running multi-container Docker applications.
 with compose, you use a YAML file to configure your applications services.
 we can start all the services with single command - docker compose up
 we can stop all the services with single command - docker compose down
 
 Need to install docker compose.
 
 docker-compose.yml
 
 version: '3'
 services:
   webserver:
     image: nginx
   database:
     image: redis
     
   docker-compose config # check the format error
   docker-compose up -d # to run in detached mode
   
   docker-compose down
    version: '3'
 services:
   webserver:
     image: nginx
     ports:
       - "8080:80"
   database:
     image: redis
   
   
 docker-compose up -d
 
version: '3.3'
 
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
 
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
 
 

 MULTI SERVICE APPLICATION DEPLOYMENT
 
 A specific web-application might have multiple containers that are required as part of the build process.
 Whenever we make use of docker service, it is typically for a single container image.
 The docker stack can be used to manage a multi-service application.
 
 DOCKER STACK
 
  A stack is a grouop of interrelated services that share dependencies, and can be orchestrated and scaled together.
  A stack can compose YAML file like the one that we define during Docker Compose.
  We can define everythign within the YAML file that we might define while creating a Docker Service
  
  docker stack deploy --compose-file docker-compose.yml mydemo
  docker stack ps mdemo
  
  docker stack rm mydemo
  
  
  LOCKING SWARM CLUSTER
Swarm cluster contains lot of sensitive information, which includes:

* TLS key used to encrypt communication among swarm node
* Keys used to encrypt and decrypt the Raft logs on disk

If your swarm is compromised and if data is stored in plain-text, an attack can get all the sensitive information.
Docker locks allows us to have control over the keys.

cd /var/lib/docker

ls -l
cd swarm
cd certificates
ls -l
cat *.key

docker swarm update --autolock=true

docker swarm unlock
docker swarm unlock-key
docker swarm unlock-key --rotate
 
 
 TROUBLESHOOTING SWARM SERVICE
 
 A service may be configured in such a way that no node currently in the swarm can run its tasks.
 In this case, the service remains in state pending
 
 There are multiple reasons why service might go into pending state
 
 * If all nodes are drained and you create a service, it is pending until a node becomes available
 
* you can reserve a specific amount of memory for a service, if no node in the swarm has the required 
amount of memory, the service remains in a pending state until a node is available which can run its tasks.
* You have imposed some kind of placement constraints

docker service create --name demotroubleshoot --constraint node.labels.region==1mumbai --replicas 3 nginx
docker service ps demotroubleshoot
docker inspect <task-name>

MOUNTING VOLUEM VIA SWARM

docker service create --name myservice --mount type=volume,source=myvolume,target=/mypath nginx
docker service ps myservice

docker volume ls
docker exec -it id /bin/bash

cd /mypath
touch test-file
exit

cd /var/lib/docker/volumes/
ls
cd _data/
ls

docker ps
docker service ls
docker servce rm myservice

docker volume ls
cd /var/lib/docker/volumes/
ls
cd _data/
ls

CONTROL SERVICE PLACEMENT

Swarm services provides a few differen ways for your to control scale and placement of services on different nodes.

- Replicated and Global Services
- Resource Constraints [requirement of CPU and Memory]
- Placemet Constraints [only run on nodes with label "pci_compliance = true"]
- Placement preferences

docker node ls
docker service create --name myconstraint --constraint node.labels.region==blr replicas 3 nginx
docker service ps myconstraint
docker node inspect node-id
docker service rm myconstraint

docker node ls
docker node update --label-add region=mumbai node-id
docker node inspect node-id
docker service create --name myconstraint --constraint node.labels.region==mumbai replicas 3 nginx
docker service ps myconstraint

OVERLAY NETWORKS

The overlay network driver creates a distributed network among multiple Docker daemon hosts
Overlay network allows containers connected to it to communicate securely.

Whenever we initiate the swarm it will create a overlay network

docker network ls

docker network inspect my-network

CREATING CUSTOM OVERLAY NETWORK

docker network create --driver overlay mynetwork
docker service create --name myoverlay --network mynetwork --replicas 3 nginx
docker node ls

docker service rm myoverlay
docker service ls 
docker network ls #custom network will be removed 

SECURE OVERLAY NETWORK

For the overlay networks, the containers can be spreaded across multiple servers.
If the containers are communicating with each other, it is recommended to secure the communication.
To enable encryption, when you create and overlay network pass the --opt encrypted flag:

When you enable overlay encryption, Docker creates IPSEC tunnels between all the nodes where tasks are
scheduled for services attached to the overlay network.

These tunnels also use the AWS algorighm in GCM mode and manager nodes automatically rotate the keys every 12 hours

Overlay network encryption is not supported on Windows. If a Windows node attempts to connect to an encrypted overlay network,
no error is detected but the node will not be able to communicate.

docker network create --opt encrypted --driver overlay my-overlay-secure-network

CREATING SWARM SERVICE USING TEMPLATE

  We can make use of templates while running the service create command in swarm.
  
  Let us understand this with an example:
  
  docker service create --name demoservice--hostname="{{.Node.Hostname}}-{{.Service.Name}}" nginx


Valid Placeholders

Placeholder      Description
Service.ID       Service ID
.Service.Name    Service Name
.Service.Labels  Service labels
.Node.Id         Node ID
.Node.Hostname   Node Hostname
.Task.ID         Task ID
.Task.Name       Task Name
.Task.Slot       Task slot

There are three supported flags for the placeholder templates:

  --hostname
  --mount
  --env
  
SPLIT BRAIN & IMPORTANCE OF QUORUM

If we have only two nodes cluster may endup with split brain so we need three node cluster quorum voting 
Need to have ODD number of nodes in cluster

CLUSTER SIZE       MAJORITY      FAULT TOLERANCE
1                   0                   0
2                   2                   0
3                   2                   1
4                   3                   1
5                   3                   2
6                   4                   2
7                   4                   3

HIGH AVAILABILITY OF SWAM MANAGER NDES


 Manager nodes are responsible for handling the cluster management tasks.
 * Maintaining cluster state
 * Scheduling services
 * Serving swarm mode HTTP API endpoints
 
 Using a Raft implementation, the managers maintain a constistent internal state of the entire swarm and all the services running on it.
 
 Swarm comes with it's own fault tolerance features.
 Docker recommends you implement an odd number of nodes according to your organization's high-availability requirements
 Using a Raft implementation, the managers maintain a consistent internal state of the entire swarm and all the services running on it
 An N manager cluster tolerates the loss of an most (N-1)/2 managers.
 
 *docker swarm join-token manager*
 
 * Maximum of 7 manager is recommended by docker
 
 MANAGER ONLY NODES IN SWARM
 
 create a simple service with 3 replicas
 
 docekr service create --name webserver --replicas 3 nginx
 docker service ps webserver
 
 docker node update --availability drain <node-id>
 
 RECOVERING FROM LOSING THE QUORUM
 
  Create a 2 master and 1 worker
  and do drain on both master
  docker node update --availability drain <node-id>
  docker service create --name webserver --replicas 3 nginx
 stop docker in one master
 systemctl stop docker
 docker node ls # it will throw error
 
 
 docker swarm init --force-new-cluster --advertise-addr
 
 
 
 DOCKER SYSTEM COMMANDS
 
 https://docs.docker.com/engine/reference/commandline/system_events
 
docker system events

stop one container
doker stop container-name

docker system events --since 2019-03-20

docker system df 

