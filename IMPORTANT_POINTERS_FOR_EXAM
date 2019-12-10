IMPORTANT_POINTERS_DOMAIN 1

PART 01

1. You should know how to create swarm service
  * docker service create --name webserver --replicas 1 nginx
  
2. Difference between replicated and global service

  * Replicated service will have N number of containers defined with the --replica flag
  * Global service will create 1 container in every node of the cluster.
  
PART 02

3. Multiple approaches to scale swarm service
  
  * docker service scale mywebserver=5
  * docker service update --replicas 5 mywebserver
  
4. Difference between two approaches to scale swarm service
  
  * docker service scale allows us to specify multiple services in same command.
  * docker service update command only allows us to specivy one service per command
  
PART 03

5. Draining Swarm Node

  * docker node update --availability drain swarm03
  * docker node update --availability active swarm03
  
6. Understanding the use-case of Docker Stack CLI

  * docker stack deploy --compose-file docker-compose.yml
  
PART 04

7. Placement Constraints

  * docker service create --name webserver --constraint node.label.region==blr nginx
  * docker service create --name webserver --constraint node.label.region!=blr nginx
  
8. Adding Labels to a node

  * docker node update --label-add region=mumbai worker-node-id
  
PART 05

9. Overlay Networks & Security

  Overlay network allows containers spreaded across different servers to communicate.
  The communicaiton between containers can be made secured with IPSec tunnels.
  
  docker network create --opt encrypted --driver overlay my-overlay-secure-network
  
 PART 06 
 
10. Revise and Double Revise the Quorum

    Cluster Size                  Majority                       Fault Tolerance
    1                              0                                 0
    2                              2                                 0
    3                              2                                 1
    4                              3                                 1
    5                              3                                 2
    6                              4                                 2
    7                              4                                 3
    
PART 07    
11. Troubleshooting aspect

  * "docker system events" provides you real-time even information from your server.
  * Service Deployment would be in pending state if node is drained.
  * Service deployments can also be in pending state due to placement constraints.
  * You can further inspect the task to see more information.
  
  
  
  IMPORTANT_POINTERS_DOMAIN 2
  
  PART 01
  
   1. Have thorough understanding about various Dockerfile options
   
     ADD
     COPY
     RUN
     ENTRYPOINT
     VOLUMES
     CMD
     HEALTHCHECK
     
 PART 02
 
   2. Understand difference between ADD and COPY 
   
     COPY allows us to copy files from local source to destination.
     ADD allows the same in addition to using URL & extraction capabilities of TAR files.
     
   3. Know difference between CMD and ENTRYPOINT
   
     * CMD can be overridden.
     * ENTRYPOINT cannot be overridden.
     
   PART 03
   
   4 Use-cases of using ADD vs wget / curl
   
     * Because image size matters, using ADD to fetch packages from remote URLs is strongly discouraged; you should use curl or wget instead.
     
       ADD https://example.com/big.tar.xz /usr/src/things/
       RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
       RUN make -C /usr/src/things all
       
       RUN makdir -p /usr/src/things \
           && curl -SL http://example.com/big.tar.xz \
           | tar -xJC /usr/src/things \
           && make -C /usr/src/things all
           
   PART 04
   
    5 Understanding the format option
    
       Format options allows us to format the output based on various criteria that we have defined with the command.
        
       docker images --format "{{.ID}}: {{.Repository}}"
       
  PART 05
    
    6 Understanding the filter option
      
       Filter option allows us to filter output based on condition provided.
       
       Sample Use-Case:
       
         * Show all the dangling images
         * Show all the images created after N image.
         * Show all the containers which are in the excited stage.
         
         
         docker images --filter "before=ubuntu"
         docker images --filter "dangling=true"
         
   PART 06
     
     7. Accessing in-secure docker registries
     
       By default, docker will not allow you to perform operation with an insecure registry.
       You can override by adding following stanza within the /etc/docker/daemon.json file
       
       {
       "insecure-registries" : ["myregistrydomin.com:5000"]
       }
       
   PART 07
   
     8. Pushing an image to a private repository
     
       In order to push an image to private repository, you will have to tage it with the DNS name..
       
      For Example:
        
        Registry Name: example.com
        docker tag ubuntu:latest example.com/myrepo:ubuntu
        
        
    PART 08
    
      9. Login to a private repository
      
        docker login example.com
        
      10. Searchability Aspect of Images
      
        * docker search nginx
        * docker search nginx --filter "is-official=true"
        
    PART 09
       
      11. Transferring Image / Container from one host to another
      
        * docker commit container-id image-name
        * docker save image-name > image-name.tar
        * docker export <containerID> > /home/export.tar
     
       When  we export a container, all the resultant imported image will be flattened.
       
       
       
    IMPORTANT_POINTERS_DOMAIN 3
    
      PART 01
      
      1. Overview of DTR Backup Process
        
        * When we backup DTR, there are certain things which are not backed.
        * User/Organization backup should be taken from UCP.
        
        
      Data                     Description                                                                   Backed up
      Raft keys               Used to encrypt communicaiton among swarm nodes and to encrypt and                yes
                              decrypt Raft logs                                                         
      membership              List of the nodes in the cluster                                                  yes
      Service                 Stacks and services stored in Swarm-mode                                          yes
      (overlay) networks      The overlay networks created on the cluster                                       yes
      configs                 The configs created in the cluster                                                yes
      Secrets                 Secrets saved in the cluster                                                      yes
      Swarm unlock key        MUST BE SAVED ON A PASSWORD MANAGER !                                              NO
      Images                                                                                                     NO
      
    PART 02
    
     2. Swarm Routing Mesh
     
       All nodes within the cluster participate in the ingress routing mesh.
       
        Sample Command:
        
         docker service create --name mywebserver --publish published=8080,target=80 nginx
     
     PART 03
      
      3. Know what namespace are all about.
      
       These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.
       
       User namespace is not enabled by default.
       
      
     PART 04
     
      4. Have clear understanding about cgroups.
      
        Control Groups (cgroups) is a Linux Kernel feature that limits, accounts for, and isolates the resource usage (CPU, Memory, disk I/O,Network, etc. )
        of a collection of processes.
        
        
          Approaches                           Description
          
          --cpu=<value>                 If host has 2 CPUs and if you set --cpu=1, than container is guaranteed at most one CPU. You can even specify --cpu=0.5
          
          --cpuset-cpus                 Limit the specific CPUs or cores a container can use. A comma-separated list of hyphen-separated range of CPUs a container can use.
                                        
                                        1st CPU is numbered 0
                                        2nd CPU is numbered 1
                                        
                                        Value of 0-3 means usage of first, second, thired and fourch CPU.
                                        Value of 1,3 means second and fourth CPU.
                                        
       PART 05
       
        5. Reservation vs Limits
        
         * Limit is hard limit.
         * Reservation is a soft limit.
         
         * docker container run -dt --name container01 --memory-reservation 250m ubuntu
         * docker container run -dt --name container02 -m 500m ubuntu
         
       PART 06
       
       6. Immutable Tags in DTR
          
          * By default, users with read and write access can overwrite tags.
          * To prevent tags from being overwritten, we can configure repository to be immutable.
          
      
      IMPORTANT_POINTERS_DOMAIN 4
      PART 01
      
        You should be aware of the primary networking drivers.
        
         1. Bridge Network Driver:
         
          * Bridge is the default network driver for Docker.
          
        
      PART 02
      
       2. Host Network
       
         * This driver removes the network isolation between the docker host and the docker containers to use the host's networking directly
         
       3. Overlay Network
       
         * Default in Swarm.
         * Allows containers across host to communicate with each other.
         * Communicaiton can be encrypted with --opt encrypted option.
         * Do not confuse, -o is smae as --opt
         
       PART 03
       
         4. User defined Bridge and Legacy Link Approach
         
           Remember that --link is a legacy approach and should not be used.
           
         5. Be aware about the --opt while creating networks
         
           docker network create --driver=bridge --subnet 172.31.0.0/16 --gateway 172.31.0.1 --ip-range 172.31.1.0/24 --opt "docker.network.driver.mtu=1540" mybro
           
           
       PART 04
       
         6. Know difference between publish list (-p) and publish all (-P)
         
           * Publish List (-p) will publish list of ports that you define. [-p 80:80]
           * Publish All (-P) will asign random ports for all exposed ports of the container.
           * -P will map the container port to random port above 32768
           
         7. None Network
           
           * If you want to completely disable the networking stack on a container, you can use the none network.
           * This mode will not configure any IP for the container and doesn't have the external network as well as for other containers.
           
        PART 05
        
         8. Configuring docker for external DNS
         
           * Docker container's DNS configuration is taken from the host's /etc/resolv/conf
           * DNS settings for containers be customized vis daemon.json file.
           
           {
             "dns": ["8.8.8.8","172.31.0.2"]
           }
           
           
    IMPORTANT_POINTERS_DOMAIN 5 SECURITY
    
    PART 01
    
     1. Docker Content Trust
     
       * Used for interacting with only trusted images (signed images)
       * Enabled with export DOCKER_CONTENT_TRUST=1
       
     2. UCP Client Bundles
     
       * Client bundles are group of certificates and files downloaded from UCP
       * Depending on the permissoin associates with the user, you can now execute the swarm commands from your remote machine that take 
         effect on the remote swarm cluster
    
     3. Docker Secrets
     
       Remember that you cannot update or rename a secret, but you can revoke a secret and grant access to it using a new target filename.
       
       docker service update --secret-rm mysql_password mysql
       docker service update --secret add source=NEW-MYSQL-PASS, target=mysql_password mysql
       
       
     4. know about the how you can lock the swarm cluster
       If your swarm is compromised and if data is stored in plain-text, an attack can get all the sensitive information.
       
       Docker Lock allows us to have control over the keys,
       
         * docker swarm update --autolock-true
    
      
      
      IMPORTANT_POINTERS_DOMAIN 6 STORAGE AND VOLUMES
      
      
      1. Mounting Volumes Inside Container
      
        * docker container run -dt --name webserver -v myvolume:/etc/ busybox sh
        * docker container run -dt --name webserver --mount source=myvolume,target=/etc busybox sh
        
          You can even mount same volume to multiple containers.
          
            also know what bind mounts are all about
            How you can map a directory from host container
            
       2. Remove volume when container exits
       
         If you have not named volume, then when you specify --rm option, the volumes associated will also be delted when container is deleted.
         
       3. Device Mapper
       
         * loop-lvm mode is for testing purpose only.
         * direct-lvm mode can be used in production.
         
      
      
       
