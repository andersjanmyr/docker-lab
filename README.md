# Docker Lab

## Install Docker

https://docs.docker.com/installation/


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

Run some commands, create a couple of files, then exit.

```
# List all containers, including stopped
$ docker ps --all

# List all running containers
$ docker ps --all

# List all container IDs
$ docker ps --all -q

# List latest container including stopped
$ docker ps -l

# Remove the latest container
$ docker rm $(docker ps -l -q)
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
