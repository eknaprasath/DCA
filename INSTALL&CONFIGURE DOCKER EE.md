Docker Enterprise Edition (also referred as Docker EE) is designed for enterprise development as well as IT teams who build,
ship and run business-critical applications in production and at scale

Docker EE is officially supported by the Docker Team.

Enterprise edition come up with Stability, Support and some unique features
 
There are multiple tiers available for Docker Enterprise edition. Referred as Basic, Standard and Advanced.

 Capabilities                    Docker Engine - community        Docker Engine- Enterprise        Docker Enterprise
 Container engine and built in         Yes                              Yes                              Yes
 orchestration, networking, 
 secuity  
 
 Certified Infrastructure,                                              Yes                              Yes 
 plugins and ISV containers
 
 Image management                                                                                        Yes
 
 Container app management                                                                                Yes
 
 Image Security Scanning                                                                                 Yes
 
 
 
 INSTALLING DOCKER EE
 
 Subscribe for EE trail 
 
 user-name--> my content --> setup --> license key
 copy the URL to configure 
 choose the os version --> copy the link -->wget 
 
 nano docker-ee.repo
 cp docker-ee.repo /etc/yum.repos.d/
 
 export DOCKERURL="INSERT-YOUR-URL-HERE-WHICH-IS-COPIED-FROM-DOCKER-CONSOLE"
 
sudo -E sh -c 'echo "$DOCKERURL/centos" > /etc/yum/vars/dockerurl'
 
yum -y install docker-ee docker-ee-cli containerd.io
 
systemctl start docker
systemctl enable docker
 
docker login
 
docker engine activate 
 
 
DOCKER UNIVERSAL CONTROL PLANE (UCP)

Docker UCP is the enterprise-grade cluster management solution from Docker.
UCP help you manage your Docker cluster and applications through a single interface.

UCP Architecture.

Universal Control Plane is a containerized application that runs on Docker EE.
Once Universal Control Plane (UCP) instance id deployed, developers and IT operations no longer interact 
  with Docker Engine directly, but interact with UCP instead.
  
System Requirements for UCP

Minimum Requirements for UCP:

* Docker EE engine.
* 8GB of RAM for Manager Nodes
* 4GB of RAM for Worker nodes
* 5 GB of free disk space for the /var partition for manager nodes
* 500MB of free disk space for the /var partiton for worker nodes

UCP INSTALLATION

docker container run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp:3.1.4 install --host-address 139.59.82.72 --force-minimums

UCP ACCESS CONTROL

  Docker UCP allows us provides granular access control feature which allows us to define various aspects like who can create and edit
  container resources in your swarm and others.
  
  "TEAM B" has "Restricted Control" on "Collection A"
  
  SUBJECTS                 ROLES                       COLLECTIONS
  Organization A          No Access                  A         B         c
  TeamA    TeamB          Read Only               Node A    Secret A    Node B
                         Restricted Control       Secret B  Stack C     Node C
                         Full Control             Service A             Service B
                                                                        Service C
                                                                        
 UNDERSTANDING SUBJECT
 
 A subject represents a user, team, or organization.
 A subnect is in-turn granted an appropriate role.
 
 User: Individual User.
 
 Organization: A group of users that share a specific set of permissions, defined by the roles of the organization.
 
 Team: A group of users that share a set of permissions defined in the team itself.
       A team exists only as part of an organization.
       
 
 UNDERSTANDING ROLES
 
   A role is a set of permitted API operations that you can assign to a specific subject and collection.
   
   
 UNDERSTANDING COLLECTIONS
 
    Docker EE enables controlling access to swarm resources by using collections.
    
    Swarm resources that can be placed in to a collection include:
    
    * Physical or virtual nodes
    * Containers
    * Services
    * Networks
    * Volumes
    * Secrets
    * Application Configs
    
    
OVERVIEW OF DOCKER TRUSTED REGISTRY

   Docker Trusted Registry (DTR) is the enterprise-grade image storage solution from Docker.
   
   DTR provides many features that helps in overall enterprise grade container management.
   Some of these features include:
   
    * Security Scanning
    * Image signing
    * Built-in access control
    * Caching Images
    
   Docker Trusted Registry (DTR) is a containerized application that runs on a Docker Universal Control Plane cluster
   
   SYSTEM REQUIREMENTS FOR DTR
   
     Minimum requirements for DTR:
     
       * 16 GB of RAM for nodes running DTR
       * 2 vCPUs for ndoes running DTR
       * 10 GB of free disk space
       
     To install DTR, all nodes must:
       Be a worker nodes managed by UCP ( Universal Control Plane )
       Have a fixed hostname


wget https://159.65.151.132/ca --no-check-certificate
cp ca /etc/pki/ca-trust/source/anchors/example.com.crt
update-ca-trust
systemctl restart docker
add host entry for the example.com
docker login example.com
docker tag busybox:latest example.com/admin/webserver:v1
docker push exam
docker push example.com/admin/webserver:v1

DTR Backup

Docker DTR has a backup command that allows us to take the backup
The backup command doesn't cause downtime for DTR, so you can take frequent backups without affecting your users.

docker run --log-driver none -i --rm docker/dtr backup --ucp-url https://172.31.40.237 -ucp-insecure-tls --ucp-username admin --ucp-password YOUR-PASSWORD-HERE > backup.tar

OVERVIEW OF ROUTING MESH

 There can be multiple nodes within a swarm cluster.
 When creating a service, the task can be assigned to nodes depending on various factors.
 Routing mesh enables each node in the swarm to accept connections on published ports for any service running in the swarm, even if there's
 no task running on the node.
 
 Ingress network
 
docker node ls

docker service ls
docker service ps my-web
get the ip address of the node where the service running

try to curl the another host where the service not running and verify the service is reachable



docker service create --name webserver --publish published=8080,target=80 --replicas 1 nginx

docker service ps webserver
netstat -tunlp ( on both servers and check the port if occupied on both servers )




