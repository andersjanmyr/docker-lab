# Docker Lab

## Requirements

* Linux
* OS X
* Windows 7.1, 8 or newer BUT NOT Windows 10.


## Install Docker

https://docs.docker.com/installation/

If you are running OSX or Windows add the IP address of the virtual machine to
`/etc/hosts` to simplify interacting with it.

```
$ sudo vi /etc/hosts # or use another editor :)
# Add the IP of the virtual machine
192.168.99.100 docker.local
# Exit and save with :wq or ZZ

# Verify that it works
$ ping docker.local
# Exit with Ctrl-c
```

## TODO

* Inspect logs
* Inspect container
* Copy files from and to a container.
* Start up a container with a database
* Link to the database container
* Docker compose
* Docker machine
* Docker swarm

## Test Images

* debian - Official Debian Image
* ubuntu - Official Ubuntu Image
* scratch - Base image for other images, 0 bytes
* busybox - Minimal Linux distro, 2.5MB
* node - Official Node.js image
* redis - Official Redis image
* mongo - Official Mongo image
* postgres - Official Postgres image
* jwilder/nginx-proxy - Nginx image with automatic proxy configuration of other
  containers.
* andersjanmyr/counter - Counter web-service with support for multiple
  databases.


## Use a simple throwaway container

```
$ docker run -it --rm busybox /bin/sh
```

* `docker run` - Start a docker container.
* `-it` - Interactive, pass stdin (-i) as a tty, pass signals (-t).
* `--rm` - Remove the container once it is stopped.
* `busybox` - A minimal docker image
* `/bin/sh` - The command to run, start a shell.


## Use a simple "persistent" container

```
$ docker run -it busybox /bin/sh
```

Same as before, but no `--rm` to remove the container once you exit.

Run some commands, create a couple of files, then exit `exit`-command or
`Ctrl-D`

```
# List all containers, including stopped
$ docker ps --all

# List all running containers
$ docker ps

# List all container IDs (-q quiet)
$ docker ps --all -q

# List latest (-l) container, including stopped
$ docker ps -l

# List ID of latest container
$ docker ps -l -q

# Start and attact to the stopped container
$ docker start -ia $(docker ps -l -q)

# Exit the container, exit or Ctrl-D

# List the container and remember its name.
# docker ps -a

# List the logs of the container by name
$ docker logs name-of-container # (drunk_perlman or similar)

# Remove the container
$ docker rm $(docker ps -l -q) # Or use the name
```



## Build an image interactively

```
# Start a debian image or (ubuntu)
$ docker run -it debian:jessie /bin/bash

# Install some stuff
...
$ exit

$ docker ps -l -q
38bbf0b3eb74

# Commit the image
$ docker commit 38bbf0b3eb74 yourname:atag

$ docker images
REPOSITORY               TAG         IMAGE ID            CREATED             VIRTUAL SIZE
yourname                 atag        06c6650f1766        3 seconds ago       2.433 MB
lifelog/nginx-confd      latest      7b181acb34c6        About an hour ago   225.9 MB
lifelog/nginx-confd      v1.0.0      7b181acb34c6        About an hour ago   225.9 MB
...
```

## Build an image with a Dockerfile

```
$ mkdir mydir
$ cd mydir
$ vi Dockerfile
$ docker build -t yourname:btag .
```

## Containers with environment variables

* Use `docker run -e VARIABLE=VALUE`
* Use `ENV` in Dockerfile
* Use `docker run --env-file env`

```
```

## Containers with ports

* Use `docker run -p 8080:80`
* Use `docker run -p 8080`
* Use `EXPOSE` in Dockerfile
* Use `docker --link`

```
```

## Containers with volumes

* Use `docker run -v $(PWD):/tmp/local`
* Use `docker run -v /tmp/whereami`
* Use `VOLUME` in Dockerfile
* Use `docker --volumes-from`

```
```

# Core OS


## Start a Core OS cluster

https://coreos.com/docs/quickstart/

## Create some more advanced services

https://coreos.com/docs/launching-containers/launching/launching-containers-fleet

## Write a service that uses etcd and launch it into the cluster

* Länka ihop med en environment variables
* Använda volumer
* Använda --link
* Använda --volumes-from
