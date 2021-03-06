* A docker image is built up from a series fo layers.
* Storage Driver piece all these things together for you
* There are multiple storage drivers availabel with different trade-off

FROM ubuntu:16.04
COPY . /app
RUN make /app
CMD python /app/app.py

THE COPY ON WRITE (CoW) STRATEGY

"Copy on write" means: everyone has a single shared copy of the same data until it's written, and then a copy is made.

                  Common file
                  
           Program 1       Program 2
           
           
  docker info #to check the storage driver
  
  cd /var/lib/docker
  ls  -l
  you can see the overlay2
  
  cd overlay2
  ls -l
  cd l
  ls
  
  docker images
  docker pull ubuntu
  cd ..
  ls -l
  
  you can see multiple directories 
  
  
           
    VARIOUS STORAGE DRIVERS AVAILABLE
    
    There are various storage drivers availabel in Docker, these includes:
    
    * AUFS
    * Device Mapper
    * OverlayFS
    * ZFS
    * VFS
    * Btrfs
    
    
    
    BLOCK VS OBJECT STORAGE
    
    Block Storage
    
      In Block storage, the data is stored in terms of blocks
      Data stored in blocks are normally read or writtend entirely a whole block at the same time
      Most of the file systems are based on block devices.


lsblk
blockdev --getbsz /dev/sdb

      Every block has an address and application can be called via SCSI call via it's address
      There is no storage side meta-data associated with the block except the address.
      Thus block had no description, no owner.
      
      
      OBJECT SOTRAGE
      
      Object storage is a data storage architecture that manages data as objects as opposed to blocks of storage.
      An object is defined as a data (file) along with all it's meta-data which is combined together as an object.
      This object is given an ID which is calculated from the content of the object (from the data and metadata). Application can then call object with the unique object ID.
      
      
      DIFFERENCE
      
      HTTP(S) INTERFACE                                       Block Storage
      Object Storage   
      
      * Store virtually unlimited files.                   File is split and stored in fixed sized blocks.
      * Maintain file revisions.                           Capacity can be increased by adding more nodes
      * HTTP(S) based interface.                           Suitable for applications which require high IOPS, database, transactional data.
      * File are distributed in different physical nodes.
      
      CHANGING STORAGE DIRVERS
      
      
      https://docs.docker.com/storage/storagedriver/select-storage-driver
      https://docs.docker.com/storage/storagedriver/select-storage-driver/
      
      
      
      docker info # check for storage driver and OS version
      
      If we change the storage dirver existing containers and data will be inaccessible 
      
      cd /var/lib/docker
      ls 
      cd overlay2
      ls
      
      
      stop the docker
      
      systemctl stop docker
      
      cd /etc/docker
      
      vi daemon.json
      
      {
       "storage-dirver": "aufs"
      }
      
      systemctl start docker
      docker info # check the storage driver now
      
      docker ps -a
      
      exising containers will not be shown
      
      
      CHALLENGES WITH FILES IN CONTAINER WRITABLE LAYER
      
       By default all files created inside a container are stored on a writable container layer. This means that:
          The data doesn't persist when that container no longer exists. and it can be difficult to get the data out of the 
          container if another process needs it.
          
      CHALLENGES WITH FILES IN CONTAINER WRITABLE LAYER - PART 2
      
       Writing into a container's writable layer requires a storage dirver to manage the filesystem.
       The storage driver provides a union filesystem, using the Linux Kernel.
           This extra abstraction reduces performance as comapared to using data volumes, which write directly to the host filesystem.
           
           
     IDEAL APPROACH FOR PERSISTENT DATA
       
        Docker has tow options for containers to store files in the host machine, so that the files are persisted even after the container sotps:
        volumes, and bind mounts.
        
        If you're running Docker on Linux you can also use a tempfs mount.
        
        
        Launch one container and go to overlay2 directory check for that
        
        cd /var/lib/docker/overlay2
        ls
        
        docker stop nginx-container
        docker rm nginx-container
        
        ls # now we can see that container directory delted 
        
           
       docker volume ls
       docker volume create myvolume
       
       docker volume ls
       
       docker container run -dt --name busybox -v myvolume:/etc busybox sh
       
       docker volume inspect myvolume # get the mount point
       
       cd /var/lib/docker/volumes/myvolume/_data
       ls -l
       
       docker container exec -it busybox sh
       
       cd /etc
       ls
       
       docker stop busybox
       docker rm busybox
       
       cd /var/lib/docker/volumes/myvolume/_data
       ls -l
       
       data will be still present in the path
       
       we can remove the volume using as follows
       
       docker volume rm myvolume
       
       
       FROM busybox
       RUN mkdir /myvolume
       RUN echo "hello world" > /myvolume/hellow.txt
       VOLUME /myvolume
       
       docker build -t mydemo .
       
       docker run -dt --name mydemo mydemo sh
       docker ps
       docker volume ls # new volume will be created 
       
   to specify the volume with name
   
   docker container run -dt --name mydemo02 -v mydemo02:/myvolume mydemo sh
   docker volume ls
   
   
   IMPORTANT POINTERS TO REMEMBER
   
     A given volume can be mounted into multiple containers simultaneously.
     When no running container is using a volume, the volume is still available to Docker and is not removed automatically.
     When you mount a volume, it may be named or anonymous. Anonymous volumes are not given an explicit name when they are first 
       mounted into a container, so Docker gives them a random name that is guaranteed to be unique within a given Docker host.
       
       
    
    BIND MOUNTS
    
      When you se a bind mount, a file or directory on the host machine is mounted into a container.
      The file or directory is referenced by its full or relative path on the host machine.
      Bind mount refered to the existing directory of the machine like "/root/docker"
      
      docker container run -dt --name nginx nginx
      docker inspect nginx # get the ip address
      curl that ip
      docker container exec -it nginx bash
      cd /usr/share/nginx/html
      cat index.html
      
      mkdir /root/index
      echo "This is index file" > index.html
      
      METHOD 1
      
      docker container run -dt --name nginxbind01 -v /root/index/:/usr/share/nginx/html nginx
      docker ps
      docker inspect nginxbind01 # get the ip address
      curl <ip address>
      
      METHOD 2
      
      docker container run -dt --name nginxbind02 --mount type=bind,source=/root/index,target=/usr/share/nginx/html nginx
      
      READ ONLY MOUNT
      
      docker container run -dt --name nginxbind03 --mount type=bind,source=/root/index,target=/usr/share/nginx/html,readonly nginx
      
       
       
   AUTOMATICALLY REMOVE VOLUMES ON CONTAINER EXISTS 
   
      Whenever a container is created with -dt flag and if the main process completes/stops, then it goes into the Exited stage.
      We can see list of all containers(Running/Exited) with "docker ps -a" command.
      Many times, containers performs a job and we want container to automatically get deleted once it exits. This can be achieved withthe --rm option.
       
   
   docker container run -dt --name container01 busybox ping -c10 google.com
   
   docker ps
   
   docker ps -a
   
   docker container run --rm -dt --name container02 busybox ping -c10 google.com
   
   docker ps 
   docker ps -a
   
   docker container run -dt --name container02 -v /testvolume busybox ping -c10 google.com
     
   docker volume ls
   docker container prune
   
   docker volume ls
   
   docker container run --rm -dt --name container02 -v /testvolume busybox ping -c10 google.com
   docker ps 
   docker ps -a
   docker volume ls
   
   OVERVIEW OF DEVICE MAPPER
   
      Device mapper creates logical devices on top of physical block device and provides feature likes: 
      
      * RAID, Encryption, CAche and various others.
      
      
      
      Logical device #001
      Logical device #002
      Logical device #003           Pointers from segments of logical devices to block in the pool are stored.
      
      Block Pool                      Metadata Device
      
   
   
   Few Importan Pointers
   
   There are two modes for devicemapper storage driver:
   
   i) loop-lvm mode:
   
     * Should only be used in testing environment.
     * Makes use of loopback mechanism which is generally on the slower side.
     
   ii) direct-lvm mode:
   
     * Production hosts using the devicemapper storage driver must use, direct-lvm
     * This is much more faster then the loop-lvm mode.
     
     
     
   vi /etc/daemon.json
   
   {
     "storage-driver": "devicemapper"
   }
   
  docker info | grep -i storge
  
  systemctl restart docker
  
  
  
  LOGGING DRIVERS
  
    
       
    UNIX and Linux commands typically open three I/O strems when they run, called STDIN, STDOUT, and STDERR
    
    
        bash                                        nginx
   stdin    stdout stderr                    stdin stdout stderr
    |                                           |
   things that you type                   Things that you type
                       |                  |
                       Your Terminal
       
      
      
      docker container run -d --name mybusybox busybox ping google.com
      
      docker ps
      
      docker logs mybusybox
      
      docker info | grep -i "logging driver"
      
      docker inspect mybusybox # check for the log file type and location
      
     
     We can change the log dirver also
     
     docker container run -d --name mycustom --log-dirver none
     
        
       
     There are a lot of logging driver options available in Docker, some of these include:
     
     * json-file
     * none
     * syslog
     * local
     * journald
     * splunk
     * awslogs
     
     The docker logs command is not available for drivers other than json-file and journald.
     
      
