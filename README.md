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

# Start and attach to the stopped container
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

# To connect to the redis database with a redis-cli you can use the same image
# Start an interactive (-it) container, that will be removed on exit (--rm),
# link it to the redis container above (--link redis), run the redis image (redis)
# override the command with redis-cli and connect to the linked redis host
# (redis-cli -h redis)
$ docker run -it --rm --link redis redis redis-cli -h redis

# For help with redis-cli, try
$ docker run -it --rm redis redis-cli --help


# Start a Mongo container
$ docker run -d --name mongo mongo

# To connect to mongo, use the same technique as to connect to redis above
# Start an interactive (-it) container, that will be removed on exit (--rm),
# link it to the mongo container above (--link mongo), run the mongo image (mongo)
# override the command with mongo and connect to the linked mongo host
# (mongo --host mongo)
$ docker run -it --rm --link mongo mongo mongo --host mongo

# For help with mongo-shell
$ docker run -it --rm mongo mongo --help


# Start a Postgres container
$ docker run -d --name postgres postgres

# To connect to postgres, use the same technique again
# Start an interactive (-it) container, that will be removed on exit (--rm),
# link it to the postgres container (--link postgres), run the postgres image (postgres)
# override the command with psql and connect to the linked psql host
# (psql --host postgres -U postgres), username -U is required
$ docker run -it --rm --link postgres postgres psql -h postgres -U postgres

# For help with psql
$ docker run -it --rm postgres psql --help


# Start an Nginx container with port published on 80 (-p) and the docker
# socket mounted as a read-only volume
$ docker run -d -p 80:80 \
  -v /var/run/docker.sock:/tmp/docker.sock:ro \
  --name nginx \
  jwilder/nginx-proxy

# Browse to the container
$ open docker

# $ Start a counter web app with a VIRTUAL_HOST environment
# variable set to memory-counter.docker and port published on 8081
$ docker run -d \
  -e VIRTUAL_HOST=memory-counter.docker \
  -p 8081:80 \
  --name memory-counter \
  andersjanmyr/counter

# Browse to the container, click the counter or the digit
$ open memory-counter.docker # or docker:8081

# $ Start a counter web app linked to redis with a VIRTUAL_HOST environment
# variable set to redis-counter.docker and port published on 8082
$ docker run -d --link redis -e REDIS_URL=redis:6379 \
  -e VIRTUAL_HOST=redis-counter.docker \
  -p 8082:80 \
  --name redis-counter \
  andersjanmyr/counter

# Browse to the container, click the counter or the digit
$ open redis-counter.docker # or docker:8082

# $ Start a counter web app linked to postgres with a VIRTUAL_HOST environment
# variable set to postgres-counter.docker and port published on 8083
$ docker run -d --link postgres \
  -e POSTGRES_URL=postgres://postgres@postgres \
  -e VIRTUAL_HOST=postgres-counter.docker \
  -p 8083:80 \
  andersjanmyr/counter

# Browse to the container, click the counter or the digit
$ open postgres-counter.docker # or docker:8083

# $ Start a counter web app linked to mongo with a VIRTUAL_HOST environment
# variable set to mongo-counter.docker and port published on 8084
$ docker run -d --link mongo \
  -e MONGO_URL=mongo:27017 \
  -e VIRTUAL_HOST=mongo-counter.docker \
  -p 8084:80 \
  andersjanmyr/counter

# Browse to the container, click the counter or the digit
$ open mongo-counter.docker # or docker:8084

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

Inspect the containers to find out the following information.

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
* What hostnames are available in `/etc/hosts`?
* Where do the hostnames come from?
* It may not be possible to get into some of the containers, why is this?
* How does the `nginx-proxy` work? Check out the nginx configuration files
  located in the container at `/etc/nginx/nginx.conf` and
  `/etc/nginx/conf.d/default.conf`

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
useful if you have containers that do not contain commands that allow you to
`exec` into them. It is also useful for build-containers, containers that are
used to build something instead of running it.

```
# Copy the /etc/hosts out of redis-counter
$ docker cp redis-counter:/etc/hosts .

# List the contents of the copied file
$ cat hosts
```

`docker cp` can also be used to put files into the container.

```
# Add a hostname to the hosts file and copy it back again
echo '74.125.232.127 tapir' >> hosts
$ docker cp hosts redis-counter:/etc/hosts
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
$ docker run -it debian /bin/bash

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

### Build an image from a Dockerfile

Here is a short snippet of commands to get you started with creating your own
image from a Dockerfile.

```
$ mkdir mydir
$ cd mydir
$ echo 'FROM: debian' > Dockerfile
$ docker build -t yourname:atag .
```

#### Exercises

* Create an image that runs a web service. Here are some useful images:
  [node](https://hub.docker.com/_/node/), [nginx](https://hub.docker.com/_/nginx/),
  [java](https://hub.docker.com/_/java/), [maven](https://hub.docker.com/_/maven/),
  [ruby](https://hub.docker.com/_/ruby/), [python](https://hub.docker.com/_/python/).
  It is often useful to start with an `ONBUILD` image, if one exists. If you
  need a service to build, here are some examples:
  - [express](http://expressjs.com/starter/hello-world.html) (node)
  - [simplehttpserver](https://docs.python.org/2/library/simplehttpserver.html)
    (python)
  - [nginx](https://github.com/docker-library/docs/tree/master/nginx#hosting-some-simple-static-content), just add a file...

* Create an image that runs as a tool. If you have no ideas of your own, try to make a
  `git` or an `ls` image. A tool image is configured like this:
  - It uses `ENTRYPOINT` to set to the command to run.
  - It uses `CMD` to set the default options.
  - An example with only entrypoint is [redis](https://github.com/docker-library/redis/tree/master/3.0)

* Create two separate build and run images. Build the artifacts in the build
  image, then copy it out of the build image and into the run image. (advanced)


## Docker Compose

`docker-compose` works similarly to `docker run`. In this exercise your task is
to start all the containers that we started with `docker run` in the beginning.

```
# Stop and remove all running containers.
$ docker rm -f $(docker ps -qa)

# Create a new empty directory and put a docker-compose.yml into it
$ mkdir mydir
$ cd mydir
$ touch docker-compose.yml
$ <edit> docker-compose.yml
```

### docker-compose.yml

`docker-compose` is a tool for managing multiple containers. The containers are
defined in a `docker-compose.yml` file.

```
# Output trimmed for readability
$ docker-compose -h
Commands:
  build              Build or rebuild services
  kill               Kill containers
  logs               View output from containers
  ps                 List containers
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services in background
  stop               Stop services
  up                 Create and start containers in foreground
```

```
A simple docker-compose.yml file
# The name at the root (redis) can be used when referring to the container
# within this file.
redis:
  image: redis
  volumes:
    - ./data/redis:/data

redis-counter:
  image: andersjanmyr/counter
  ports:
    - "80"
  environment:
    - REDIS_URL=redis:6379
    - VIRTUAL_HOST=redis-counter.docker
  links:
    - redis:redis
```

The configuration format is thoroughly described at the [docker-compose.yml
page at docker.com](https://docs.docker.com/compose/yml/).

When working with `docker-compose` it is frequently necessary to remove and
recreate all the containers. This is done with `docker-compose stop`, then
`docker-compose rm`. Remember that `docker-compose up` is needed to recreate
the containers.

It is simple to use `docker-compose` to interact with all the containers at the
same time, but it can also be used to interact with single containers. To get
the logs from the nginx container use `docker-compose logs nginx`

It is also possible to use the normal `docker` command to interact and debug
individual containers. Just notice that the name of the containers are
prefixed. `docker -it exec` is an invaluable tool for checking out what is
going on in the containers.


### Exercises

* Kill and remove all existing containers, `docker rm -f $(docker ps -qa)`.
* Create a `docker-compose.yml` similar to the example above, that starts all
  the containers from above, `redis`, `postgres`, `mongo`, `nginx`,
  `memory-counter`, `redis-counter`, `mongo-counter`, and `posrgres-counter`.
* List the logs from specific containers with `docker-compose <name>`
* Scale the counter containers, example `docker-compose scale redis-counter=2`
* If it didn't work, what can you do to make it work?
* When the scaling works, check out the nginx config,
  `/etc/nginx/conf.d/default.conf`. What has happened?


## Docker Swarm and Docker Machine

`docker-swarm` is the Docker solution to manage a cluster of machines. It works
by treating the cluster of machines as if it was a single machine. The simplest
way to bring a Swarm is to use `docker-machine`.

`docker-machine` is a common tool for setting up Docker enabled machines in a
lot of different environments. The currently supported environments are

* Amazon Web Services
* Microsoft Azure
* Digital Ocean
* Exoscale
* Google Compute Engine
* Generic
* Microsoft Hyper-V
* OpenStack
* Rackspace
* IBM Softlayer
* Oracle VirtualBox
* VMware vCloud Air
* VMware Fusion
* VMware vSphere

Take special notice of the `Generic` driver in this list as this can be used to
configure Docker on any machine, such as local hardware.

In this lab we will continue to use VirtualBox, but now we will bring up some
more machines.


### Start Swarm Machines

Spin up a swarm with three machines.

```
# Create and save a token, using the Docker-Hub discovery service, default
$ token=$(docker run swarm create)

# Create a swarm manager using the token
$ docker-machine create \
  -d virtualbox \
  --swarm \
  --swarm-master \
  --swarm-discovery token://$token \
  swarm-master

# Create a new node named frontend and label it public
$ docker-machine create \
  -d virtualbox \
  --swarm \
  --swarm-discovery token://$token \
  --engine-label public=yes \
  frontend

# Create two more nodes named backend1 and backend2, with no labels
$ docker-machine create \
  -d virtualbox \
  --swarm \
  --swarm-discovery token://$token \
  --engine-label public=no \
  backend1

$ docker-machine create \
  -d virtualbox \
  --swarm \
  --swarm-discovery token://$token \
  --engine-label public=no \
  backend2

# List your machines
$ docker-machine ls

# You can connect to any machine with docker-machine ssh <machine-name>
$ docker-machine ssh frontend

docker@swarm-frontend:~$ exit
```

### Take control of the swarm

```
# List the environment needed to connect to the swarm
$ docker-machine env --swarm swarm-master

# Configure docker to use them
$ eval $(docker-machine env --swarm swarm-master)

# List information about the cluster
$ docker info
```

### Deploy your containers to the swarm


Now it is time to deploy. Try to deploy the `docker-compose.yml` file and see
what happens.

Tweak the `docker-compose.yml` file to deploy to the cluster


## Used Images

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


## TODO

* Quick reference docker run
* Quick reference docker build/Dockerfile
* Docker compose
* Docker machine
* Docker swarm


