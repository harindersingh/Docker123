## Docker123
# Experimenting with Docker

# Resources used
* https://www.docker.com/products/docker-toolbox#/tutorials
* https://www.tutorialspoint.com/docker/index.htm

# Topics left
* Docker Machine
* Docker Swarm
* Docker Compose

# Install and verify if docker is installed properly
```
wget -qO https://get.docker.com/ | sh
sudo docker run hello-world
sudo usermod -aG docker <user>
```

```
docker version
docker images
```

```
Sudo docker run [options] [image] [command] [args]
Docker run -i(tells docker to connect to STDIN on the container) -t(specifies to get a pseudo-terminal) ubuntu:latest /bin/bash
```

Exit the container without actually stopping the container -> Ctrl +  P + Q

A docker has two IDs:
* Short ID
* Long ID 

Display all the running and stopped dockers:
```
docker ps -a
```

Running docker in detached mode (or as a daemon or background process):
```
docker run -d centos:7 ping 127.0.0.1 -c 100
```

Display the logs of a running or stopped container:
```
Docker logs [shortID]
```

Create a container using the tomcat image, run in detached mode and map the tomcat ports to the host port
```
docker run -d -P tomcat:7
```


# Building Images
1) Image Layers
2) The container writable layer

# Docker Commit
* saves changes in a container as a new image
* docker commit [options] [container ID] [repository:tag]
* repository name should be based on username/application
* can reference the container with container name instead of ID
* ex:docker commit [ID] panda/myapplication:1.0

```
docker run -it ubuntu:latest /bin/bash

apt-get -qq update
apt-get install curl
docker ps -a
docker commit <container ID> <yourname>/curl:1.0
docker images
docker run -it <yourname>/curl:1.0 bash
```

# Dockerfile
* is a configuration file that contains instructions for building a Docker image
* FROM instruction specifies what the base image should be
* RUN instruction specifies a command to execute
* EX:
```
FROM ubuntu:latest
RUN apt-get install vim
RUN apt-get install curl
```

* each RUN instruction will execute the command on the top writable layer and perform a commit of the image 

# Docker Build
* docker build [options] [path]
* docker build -t [repository:tag] [path]
* EX:
 ``` docker build -t pisceanpanda/myimage:1.0 . ```

# CMD Instruction
* defines a default command to execute when a container is created
* performs no action during image build
* can only be specified once in a docker file
* EX: ``` CMD ["ping", "127.0.0.1", "-c","10"] ```
* container argument overrides the CMD instruction 

# ENTRYPOINT

# MANAGING IMAGES AND CONTAINERS
```
docker ps -a
docker start <container ID>
docker stop <container ID>
docker rm <container ID>
```
```
docker images
docker rmi [image ID]
docker rmi [repo:tag]
```

# Publishing Images
```
docker login
docker tag [local repo:tag] [Docker Hub repo:tag]
docker push [repo:tag]
```

# VOLUME
* is a designated directory in a container, which is designed to persist data, independent of the container's life cycle
* volume changes are excluded when updating an image
* persist when a container is deleted
* can be mapped to a host folder
* can be shared between containers

# Mount Volumes
* can be mounted when creating or executing a container
* can be mapped to a host directory
* volume paths specified must be absolute
* Execute a new container and mount the folder /myvol into its FS
``` docker run -d -P -v /myvol nginx:1.7 ```
* Execute a new container and map the /data/src folder from the host into the /test/src folder in the container
``` docker run -i -t -v /data/src:/test/src nginx:1.7 ```

# Volumes in Dockerfile
* VOLUME instruction creates a mount point
* can specify arguments in JSON array or string
* cannot map volumes to host directories
* volumes are initialized when the container is executed
* String example
``` VOLUME /myvol ```
* JSON example
``` VOLUME ["myvol1", "myvol2"] ```

# Uses of volume
* decouple the data that is stored from the container which created the data
* good for sharing data b/w containers
  can setup a data container which has a volume you mount in other containers
* mounting folders from the host is good for testing purposes but generally not recommended for production use 

# Container Networking Features
# Mapping Ports:
* containers have their own n/w and IP address
* map exposed container ports to  ports on the host machine
* ports can be manually mapped or auto mapped
* uses -p and -P parameters in the docker run
* EX: 
``` docker run -d -P 8080:80 nginx:1.7 ```
   -P option automatically maps exposed ports in the container to a port number in the host
* only works for ports defined in the EXPOSE instruction

# Linking Containers
* linking is a communication method between containers which allows the mt o securely transfer data from one to another.
* recipient containers have access to data on source containers
* links are established based on container names

# Creating a Link
* create the source container first
``` docker run -d --name databasedocker postgres ```
* create the recipient container and use the --link option
``` docker run -d -P --name website --link databasedocker:db nginx ```

# Uses of Linking
* containers can talk to each other without having to expose ports to the host
* essential for micro service application architecture
* Example:
1) container with tomcat running
2) container with MySQL running
3) application on tomcat needs to connect to MySQL

# Docker in Continuous Integration
* CI server builds docker image and pushes into docker hub
* docker hub auto build
1) detects commits to source repository and builds the image
2) container is run during image build
3) testing done inside container
