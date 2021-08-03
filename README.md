# Docker

## Before We Begin
To speed things up, run one of the following commands to pre-download the images on to your machine:

```
docker pull ubuntu; docker pull mysql; docker pull wordpress; docker pull nginx
```
or,
```
for i in "ubuntu" "mysql" "wordpress" "nginx"; do docker pull $i; done
```

## Resources (Seriously)

https://www.youtube.com/results?search_query=docker

https://www.google.com/search?q=docker+tutorial

## What is Docker?

https://pointful.github.io/docker-intro/#/

Docker is essentially an Object Oriented Programming (OOP) for system programming. The software or application is packaged inside a **docker image** (class), which will be run (instantiated) as a **container** (object). Thus, **docker image** is a blueprint of the **docker container**. 

## Hello World (Of Course)

```bash
# Pull an image from the central image registry
# docker pull image (same as docker pull image:latest)
# docker pull image:tag
docker pull hello-world
docker pull hello-world:latest


# List images
docker images
docker image ls

# Remove image
docker rmi hello-world
docker image rm hello-world

# Remove multiple images
# docker rmi image1 image2 ...

# Run image
# docker image can be run without pulling it first. Docker engine will 
# attempt to pull it from central registry if it does not exist locally.
docker run hello-world
docker container run hello-world

# List running containers
docker ps
docker container ls

# List all containers including the ones that exit
docker ps -a
docker container ls -a

# Remove exit container
docker rm CONTAINER_NAME
docker container rm CONTAINER_NAME

# Force remove running container
docker rm -f CONTAINER_NAME
docker container rm -f CONTAINER_NAME

# Name container when starting container using --name
docker run --name hw hello-world

# Remove container by name
docker rm hw

# Remove container by container id
docker rm CONTAINER_ID

# Run container and auto-remove it on exit
docker run --rm hello-world

```

## Try Ubuntu Image

```bash
# Basic usage which does nothing
docker run ubuntu

# Run an ubuntu container in an interactive mode with TTY with -i -t or -it
# By default, it run sh shell if not specified
docker run -it --rm ubuntu

# Run bash in ubuntu, but you probably cannot tell the difference because bash
# is another shell
docker run -it --rm ubuntu bash

# Try running with echo command, it will exit immediately because echo exits
# once done so -it does nothing.
docker run -it --rm ubuntu echo "Hello World"
# Same as above without -it
docker run --rm ubuntu echo "Hello World"

# IMPORTANT: Docker container exits as soon as there is no process left 
# running inside the container.


# Sometime we have a process that can run in background and we would like
# to run docker in a detached mode. This can be done with -d option.
# The detached mode is when you would like docker engine to run the container
# in background and you can close the terminal without closing the docker
# container. This is useful when you run a server application.
#
# First, let's explore touch and tail command:
# - touch is used to create a file
# - tail is used to see the lines at the end of file
# - tail -f is used to follow/watch the file changes, it is useful to track
#           the log file.
docker run -d --name tailer ubuntu sh -c "touch hello.log && tail -f hello.log"

# now the container will be run forever until we kill it, so let's open a bash 
# terminal inside the tailer container to do something. Option -it is important.
# docker exec is used to execute stuffs inside the running container. You cannot
# use it to run something if the container is in exit state.
docker exec -it tailer bash

# This is used to copy things in/out of the container
docker cp somefile.txt tailer:newfile.txt
docker cp somedir tailer:newdir

```

## Try NGINX Image

https://hub.docker.com/_/nginx

```bash
# Run an nginx web server in a detached mode and set the restart policy to always.
# This will make sure that if the docker container stops, it will be restarted,
# which is useful for server application that you always to keep it running.
# But this restart policy will not take effect if the container is stopped
# manually.
#
# -p option is used to map the port: 
# usage: -p [host-port]:[container-port] --> this will map the container port to
#                                            host port on all host network interfaces.
# usage: -p [host-nic-ip]:[host-port]:[container-port] --> this will map port to
#                                            specific network IP
docker run --restart=always -d --name static-web -p 80:80 nginx

# Map muiltiple ports is possible
docker run --restart=always -d --name static-web -p 80:80 -p 8080:80 nginx

# Update restart policy to `no`
docker update --restart no static-web

# Investigate the container logs
docker logs static-web
# Investigate the container logs in follow mode
docker logs -f static-web

# Map volume inside docker to host directory using -v so that we can make changes
# without going into the container. This is also useful for persisting data.
# usage: -v /path/to/host:/path/in/container
#
# This map $(pwd)/data to /usr/share/nginx/html in read-only mode, so you cannot
# modify the content of /usr/share/nginx/html from inside of the container.
docker run -v $(pwd)/data:/usr/share/nginx/html:ro --restart=always -d --name static-web -p 80:80 nginx

# Start stop container by name so it is useful to name the container
docker start static-web
docker stop static-web
```

## Try MySQL Image

https://hub.docker.com/_/mysql

```bash
# Basic usage with environment variable -e, this allows you to parameterize the 
# container.
docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=password -d mysql:8.0.26

# All mysql data are persisted at /var/lib/mysql so we can map the path to host
# directory so that even if we remove the container the data remains on host
# file system.
docker run -v /my/own/datadir:/var/lib/mysql --name my-mysql -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 -d mysql:8.0.26

docker volume create xxx

docker volume ls

docker run with volume


```

## Try Wordpress with MySQL

https://hub.docker.com/_/wordpress

Scenario 1: Specify the database

```bash
docker run --name some-wordpress -e WORDPRESS_DB_HOST=10.1.2.3:3306 \
    -e WORDPRESS_DB_USER=... -e WORDPRESS_DB_PASSWORD=... -d wordpress
```

Scenario 2: Create new network (Best practice)
```bash
# Create a new docker network
docker network create wp-network
# List all docker networks
docker network ls

# Create persistance volume
docker volume create wp_db
docker volume create wp_wordpress

# Run mysql on the wp-network
docker run -d --network=wp-network \
          --name wp-mysql \
          -v wp_db:/var/lib/mysql \
          -e MYSQL_DATABASE=wordpress \
          -e MYSQL_USER=wp \
          -e MYSQL_PASSWORD=wppass \
          -e MYSQL_ROOT_PASSWORD=password \
          mysql

# Inspect docker container
docker inspect wp-mysql

# Run wordpress on the wp-network
docker run -d --network=wp-network \
    --name wp \
    -e WORDPRESS_DB_HOST=wp-mysql:3306 \
    -e WORDPRESS_DB_NAME=wordpress \
    -e WORDPRESS_DB_USER=wp \
    -e WORDPRESS_DB_PASSWORD=wppass \
    -v wp_wordpress:/var/www/html \
    -p 8080:80 \
    wordpress

```

## Let's build our own image

https://docs.docker.com/engine/reference/builder/



## Video Call App Example

https://github.com/fmeringdal/nettu-meet


## Homework 1 - Build Your Own Hello World Image

Make a docker image which will print out your own hello world message.

## Homework 2 - Build Your Own MySQL Image
Start from an ubuntu image and may your own MySQL image. Find a way to parameterize the container using enviroment variables. So oneone should be able to start a mysql with the following command:

```bash

docker run -e MYSQL_DATABASE=wordpress \
           -e MYSQL_USER=wp \
           -e MYSQL_PASSWORD=wppass \
           -e MYSQL_ROOT_PASSWORD=password \
           mymysql
```



## Homework 3 - Multi Stage Build
Explore multi stage build in docker. 


