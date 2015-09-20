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

## Running Containers

In this section you will learn how to start and stop containers.

### Use a simple throwaway container

```
$ docker run -it --rm busybox /bin/sh
```

* `docker run` - Start a docker container.
* `-it` - Interactive, pass stdin (-i) as a tty, pass signals (-t).
* `--rm` - Remove the container once it is stopped.
* `busybox` - A minimal docker image
* `/bin/sh` - The command to run, start a shell.


### Use a simple "persistent" container

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

### Start some database containers

In this section you will start up some containers that you can work with in the
following sections.

```
# Start a couple of detached (-d) Redis containers
$ docker run -d --name redis redis
$ docker run -d redis::TODO

# Start a couple of detached (-d) Mongo containers
$ docker run -d --name mongo mongo
$ docker run -d mongo:TODO

# Start a detached (-d) Postgres container
$ docker run -d --name postgres postgres

# Check that the containers are running
$ docker ps
```

## Inspecting Containers

In this section you will learn how to find information about the running
containers with the commands `docker logs`, `docker inspect`, and `docker
exec`.

### Check the logs

Most Docker images are configured to log to stdout. This is extremely useful
since it makes it very easy to check the logs of different images with  a simple
command, `docker logs`.

```
# Find out the name of the running containers
$ docker ps

# Check the logs of some of the running database containers

# List the logs of the Redis image
$ docker logs redis

# Follow (-f) the logs of the Mongo database
$ docker logs -f mongo

# List the logs of the Postgres images with docker timestamps (-t)
$ docker logs -t postgres
```

### Inspect the containers

Containers, both running and stopped, can be inspected with `docker inspect`.
`docker inspect` lists information about the container in a JSON format. Since
the command outputs *a lot* of information it can be useful with some
additional tools for finding the information.

* grep - for finding the information when you don't know the exact path.
* [jq](https://stedolan.github.io/jq/) - for flexible JSON parsing and pretty-printing.

```
# Inspect the Redis container
$ docker inspect redis

# Get the hostname of the Redis container
$ docker inspect --format '{{.Config.Hostname}}' redis

# Find everything with Host in the name with `grep`
# This is useful if you don't know exectly what you are looking for
$ docker inspect redis | grep Host

# Add a few lines of context to help figure out the path to it
$ docker inspect redis | grep Host -C2

# List the entire Config object with `json`
$ docker inspect --format '{{json .Config}}' redis

# Add `jq` to pretty-print the JSON
$ docker inspect --format '{{json .Config}}' redis |jq

# Or use `jq` without format
# Note that you have to start with .[] to get into the outer array
$ docker inspect redis |jq .[].Config
```

It is useful to learn more about the information provided by `docker inspect`.
Here's a small guide to the most interesting parts of the output.

* `Name` - the name of the container.
* `Path` - the command that started the container.
* `Args` - the arguments to the command.
* `Config` - the containers view of the configuration, a merge of data from the
  Dockerfile, and values that have been added on the command line, such as
  `--env`, `--workdir`, `--tty`, etc.
* `HostConfig` - the hosts view of the container. This maps well to the host
  options given at the command-line, such as `--publish`, `--memory`,
  `--privileged`, `--cpu*`, `--volume`, `--link`,  etc.
* `Networking` - Networking settings, `IPAddress`, and `Ports` are particularly
  useful.
* `Mounts` - the mounted volumes
* `State` - the state of the container, running, stopped, etc.




#### Exercises

Inspect the database containers to find out the following information.

* What environments variables, `Env`, are available in the containers?
* What ports are exposed, `ExposedPorts`, in the containers?
* What command, `Cmd` and `EntryPoint`, is configured to start the container?
* What command, `Path` and `Arg`, is used to start the container?
* What volumes, `Mounts`, are mounted in the containers?


### Getting inside the Containers with docker exec



## Build Images


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
