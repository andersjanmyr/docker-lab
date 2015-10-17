# Docker Lab

## Requirements

* Linux
* OS X
* Windows 7.1, 8 or newer BUT NOT Windows 10.


## Install Docker

https://docs.docker.com/installation/

If you are running OSX or Windows add the IP address of the virtual machine to
`/etc/hosts` to simplify interacting with it.
192.168.99.100 docker
```

$ sudo vi /etc/hosts # or use another editor :)
# Add the IP of the virtual machine
192.168.99.100 docker
# Exit and save with :wq or ZZ

# Verify that it works
$ ping docker
# Exit with Ctrl-c

# If ping works, also add these to /etc/hosts. They will be used later in the lab
192.168.99.100 memory-counter.docker
192.168.99.100 redis-counter.docker
192.168.99.100 mongo-counter.docker
192.168.99.100 postgres-counter.docker
```

## TODO

* Quick reference docker run
* Quick reference docker build/Dockerfile
* Dockerfiles lab
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

### Start some containers

In this section you will start up some containers that you can work with in the
following sections.

```
# Start a detached (-d) Redis container named redis with its /data volume
# mounted on directory ./data/redis
$ docker run -d --name redis -v $PWD/data/redis:/data redis

# Start a Mongo container with it's /data/db volume mounted on ./data/mongo
$ docker run -d --name mongo -v $PWD/data/mongo:/data/db mongo

# Start a Postgres container
$ docker run -d --name postgres postgres

# Start an Nginx container with port published on 8080 (-p) and the docker
# socket mounted as a read-only volume
$ docker run -d -p 8080:80 \
  -v /var/run/docker.sock:/tmp/docker.sock:ro \
  --name nginx \
  jwilder/nginx-proxy

# Browse to the container
$ open docker:8080

# $ Start a counter web app linked to redis with a VIRTUAL_HOST environment
# variable set to redis-counter.docker and port published on 8081
$ docker run -d \
  -e VIRTUAL_HOST=memory-counter.docker \
  -p 8081:80 \
  --name memory-counter \
  andersjanmyr/counter

# $ Start a counter web app linked to redis with a VIRTUAL_HOST environment
# variable set to redis-counter.docker and port published on 8082
$ docker run -d --link redis -e REDIS_URL=redis:6379 \
  -e VIRTUAL_HOST=redis-counter.docker \
  -p 8082:80 \
  --name redis-counter \
  andersjanmyr/counter

# $ Start a counter web app linked to postgres with a VIRTUAL_HOST environment
# variable set to postgres-counter.docker and port published on 8083
$ docker run -d --link postgres \
  -e POSTGRES_URL=postgres://postgres@postgres \
  -e VIRTUAL_HOST=postgres-counter.docker \
  -p 8083:80 \
  andersjanmyr/counter

# $ Start a counter web app linked to mongo with a VIRTUAL_HOST environment
# variable set to mongo-counter.docker and port published on 8084
$ docker run -d --link mongo \
  -e MONGO_URL=mongo:27017
  -e VIRTUAL_HOST=mongo-counter.docker \
  -p 8084:80 \
  andersjanmyr/counter

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

It can be very useful to get inside a running container to examine the files
inside it to see how it is working. `docker exec` lets you do this.

```
# Start a bash shell inside the Mongo container
$ docker exec -it mongo bash

# Get the environment variables of the Postgres container
$ docker exec postgres env

# List the contents of the file that is starting the Redis container
$ docker exec redis cat /entrypoint.sh
```

#### Exercises

`exec` into the running containers to find out:

* What environments variables, `env`, are set in the containers?
* What files, `ls`, are in the working directory of the containers?

### Getting files out of a container

To find out what files are in a container it is sometimes useful to export the
files to a tar-archive.

```
# Export files of Redis container to an archive
$ docker export redis > redis.tgz

# List the files in the archive
$ tar tzf redis.tgz
```

If you know the name of a file, you can copy it out of the container. This is
useful if you have build-containers, containers that are used to build
something instead of running it.

```
# Run a docker container that builds the files in /src and outputs into /dist
$ docker run --name builder -v .:/src andersjanmyr/build-binary

# Copy the binary to the current directory
$ docker cp builder:/dist/binary .
```

`docker cp` can also be used to put files into the container.

```
# Copy a config file into the nginx container
$ docker cp nginx.conf nginx:/etc/nginx/nginx.conf
```


## Dockerfiles

Now it is time to take a look at some Dockerfiles. Read the Dockerfiles of the
following images and answer the questions about them.

* [Postgres](https://hub.docker.com/_/postgres/)
* [Redis](https://hub.docker.com/_/redis/)
* [nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/~/dockerfile/)
* [Counter](https://github.com/andersjanmyr/counter/blob/master/Dockerfile)

### Questions

* What image is the Dockerfile based on?
* What environment variables does it set?
* What ports does it expose?
* What volumes does it mount?
* What command starts the container?


## Building Images

In this section you will learn how to build your own images. Images can be
created both interactively or with a Dockerfile. 


### Build an image interactively

Building an image interactively is good for experimentation. It allows you to
run commands inside a running container and then just save the image. Once you
have figured out what commands are needed for your image it is better to create
the image from a Dockerfile.

#### Exercise

Build an image interactively then commit it with a tag.

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

# List your new image
$ docker images
REPOSITORY               TAG         IMAGE ID            CREATED             VIRTUAL SIZE
yourname                 atag        06c6650f1766        3 seconds ago       2.433 MB
...
```

### Build an image with a Dockerfile

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
