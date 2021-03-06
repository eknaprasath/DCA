
Restart Policy Options

--restart flag
"no"  Do not automatically restart the container,(Default one )
"on-failure" Restart the container if it exits due to an error, which manifeasts as a non-zero exit code
"unless-stopped" Restart the container unless it si explicitly stoped or Docker itself is stopped or restarted.
"always" Always restart the container if it stops


2. Disk usage metrics for Docker components

overview of disk utilization
docker system df

disk utilization for each container and image
docker system df -v



Use CURL and WGET whenever possible

ADD http://example.com/file.tar.gz /usr/src/things/
RUN tar -xJf /usr/src/things/file.tar.gz -C /usr/src/things/
RUN make -C /usr/src/things/ all


RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/file.tar.gz \
	| tar -xJC /usr/src/things \
	&& make -C /usr/src/things all
	
HEALTHCHECK

FROM busybox
HEALTHCHECK --interval=5s CMD ping -c 172.17.0.2

--interval=DURATION (default: 30s)
--timeout=DURATION (default: 30s)
--start-period=DURATION (default: 0s)
--retries=N (default 3)

	
ENTRYPOINT

The best use for ENTRYPOINT is to set the image's main command
ENTRYPOINT doesn't allow you to override the command

	
Commiting container

  Whenever you make changes inside the container, it can be useful to commit a container's
file changes or settings into a new image.
  By default, the container being commited and its process will be paused while the image is committed 

docker container commit CONTAINER-ID myimage  
docker container commit --change='CMD ["ash"]' busybox

  Supported Dockerfile instruction:
  CMD ENTRYPOINT ENV EXPOSE LABEL ONBUILD USER VOLUME WORKDIR


UNDERSTANDING LAYERS

  * A Docker image is build up froma series of layers.
  * Each layer represents an instruction in the image's Dockerfile
  * The major difference between a container and an image is the top writable layer
  * All writes to the container that add new or modify existing data are stored in this writable layer
  
  Docker build and verify layers and size
  
  FROM ubuntu:16.04
  COPY . /app
  RUN make /app
  CMD python /app/app.py
  
 To check the layers of a container image 
    docker history <image name>
    
  Docker build and verify layers and size
  
  FROM ubuntu
  RUN dd if=/dev/zero of=/root/file1.txt bs=1M count=100
  RUN dd if=/dev/zero of=/root/file2.txt bs=1M count=100
  RUN rm -f /root/file1.txt
  RUN rm -f /root/file2.txt
  RUN apt-get update
  RUN apt-get install nano -y
  RUN apt-get install vim -y
  
  Docker build and verify layers and size
  
    FROM ubuntu
    RUN dd if=/dev/zero of=/root/file1.txt bs=1M count=100 && rm -f /root/file1.txt
    RUN dd if=/dev/zero of=/root/file2.txt bs=1M count=100 && rm -f /root/file2.txt
    RUN apt-get update && apt-get install nano -y && apt-get install vim -y
  
  
  Docker Images CLI
  
  Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  
  A Docker Image contains lots of information, some of these include:
  
    * Creation Date
    * Command
    * Environment Variables
    * Architecture
    * OS
    * Size
    
    "docker image inspect" command allows us to see all the information associated with a docker image
    
    docker image inspect nginx | grep -i hostname
    or 
    docker image inspect nginx --format='{{.Id}}'
    docker image inspect nginx --format='{{.ContainerConfig}}' #This command will get the whole portion of the output value
    docker image inspect nginx --format='{{json .ContainerConfig}}' #This command will get the Key with Value
    docker image inspect nginx --format='{{.ContainerConfig.Hostname}}'
    
    
    IMAGE PRUNING STEPS
    
    "docker image prune" command allows us to clean up unused images.
    By default, the above command will only clean up dangling images
    Dangling Images= Images without Tags and Image not referenced by any container
    
    "docker imgge prune" # it will remove the images which is not tagged
    "docker image prune -a" # it will remove the images which is not refered with any container


CONVERTING IMAGE TO SINGLE LAYER

  In a generic scenario, the more the layers an image has, the more the size of the image
  some of the image size goes from 5GB to 10GB
  Flattening an image to singel layer can help reduce the overall size of the image.
  
  docker container run -dt --name myubuntu ubuntu
  
  One of the way to create a single layer image is export and import
  
  "docker export myubuntu > myubuntudemo.tar"
  "cat myubuntudemo.tar | docker import - myubuntu:latest"
  
  docker image ls 
  docker image history myubuntu
  
  
  DOCKER REGISTRY
  
   A Registry a stateless, highly scalable server side application that stores and lets you distribute Docker images.
   
   Docker Hub is the simplest example that all of us must have used.
   
   There are various types of registry available, which includes:
   
   * Docker Registry
   * Docker Trusted Registry
   * Private Repository ( AWS ECR )
   * Docker Hub
   
   "docker run -d -p 5000:5000 --restart always --name registry registry:2"
   
   To push image need to tage the image
   
   docker tag ubuntu:latest localhost:5000/myubuntu
   
   docker image ls
   
   docker push localhost:5000/ubuntu
   
   docker pull localhost:5000/ubuntu
   
   CENTRAL REPOSITORY DOCKER HUB
   
   docker login
   
   docker tage busybox <username>/mydemo:v1
   docker push <username>/mydemo:v1
   
   docker logout
   
   
   APPLYING FILTER FOR DOCKER IMAGES
   
     docker search nginx
     
     docker search nginx --limit 5 # this will list only top 5 images based on the popularity
     
     docker search nginx --fileter "is-official=true"
  
