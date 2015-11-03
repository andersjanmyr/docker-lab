![](http://media.bestofmicro.com/I/V/468535/gallery/docker-logo_w_195.png)
# Docker CLI Quick Reference

Docker CLI Reference Docs [https://docs.docker.com/reference/commandline/cli/](https://docs.docker.com/reference/commandline/cli/)

### docker build -t repository/image-name .

Build an image with instructions from `./Dockerfile` and tag it with
`repository/image-name`.

### docker commit CONTAINER -t repository/image-name

Create a new image from a running container, the image will not include
any data from volumes.

### docker cp path1 path2

Copy `path1` to `path2`. If the paths are of the form `CONTAINER:PATH` they
will be treated as inside the `CONTAINER`.

### docker create [OPTIONS] IMAGE

Create the container but don't start it. Similar to `docker run`, but without
the start.

### docker exec CONTAINER COMMAND

Execute a command inside a running container. Useful for debugging and
inspecting inside a container. Example: `docker -it exec my-cont bash`

### docker images

List all local images.

### docker info

Shows information about the docker system.

### docker inspect IMAGE|CONTAINER

Show all relevant information about a CONTAINER or IMAGE.

### docker kill CONTAINER

Send a `--signal (-s)`, default KILL, to a CONTAINER.

### docker logs CONTAINER

List the logs from the CONTAINER. `--follow (-f)` keep file open and listen to
changes. `--tail=100` list the last 100 lines of the log, can be combined with
`--follow`.

### docker ps

List all running containers. `--all (-a)` include stopped containers, `--quiet
(-q)` list only container IDs, `--latest (-l) show only latest created
container.

### docker pull/push IMAGE

Download or upload IMAGE from/to Docker Hub.

### docker rm CONTAINER / rmi IMAGE

Remove a stopped CONTAINER/IMAGE. Running containers can be removed with
`--force (-f)`.

### docker run IMAGE

Create and start a new container base on image. See separate reference card.

### docker start CONTAINER

Start stopped CONTAINER









![logo](https://camo.githubusercontent.com/5b45fa2bae9e221a4810bb4b3999055e3929229d/687474703a2f2f7777772e6172637469717561746f722e636f6d2f6f7070656e6b616c6c6b6f642f6173736574732f696d616765732f6a61797761795f6c6f676f2e706e67)
*anders.janmyr@jayway.com*
