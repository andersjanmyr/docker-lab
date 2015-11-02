![](http://media.bestofmicro.com/I/V/468535/gallery/docker-logo_w_195.png)
# Docker Run Quick Reference

Docker Run Reference Docs [https://docs.docker.com/reference/run](https://docs.docker.com/reference/run)

## docker run

```sh
$ docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]
```

* `IMAGE` is the image name, with or without a namespace. The only images
  without a namespace are official images such as: *debian, node, redis*, etc.
  All other images have namespaces such as: *andersjanmyr/counter* or
  *ansible/centos7-ansible*.
* `TAG` is any tag that exists for the given name. *latest* is used if not tag is
  given.
* `COMMAND` is any command that the image supports, it defaults to the `CMD`
  given in the Dockerfile.

### `-d` (detached)

Start the container in the background. If you do not start it in the background
you will, most likely, need to give option `-it` to allocate a tty for the
container.

### `-it` (keep STDIN open, with a pseudo-tty)

This allows you to interact with the container, to send input into it and to
send signals such as `Ctrl-C` to stop it. `-t` should not be given if STDOUT is
redirected or piped.

### `--name` (name the container)

This name is useful for referring to the container on the CLI. By default, it
will be given a name such as *focused_feynman*.

### `--rm` (remove the container on exit)

Removes the container on exit, commonly used when running containers as tools.

### `-e` and `--env` (set an environment variable)

* `-e NAME=my-name` sets NAME to "my-name" in the container
* `-e USER` is the same as `-e USER=$USER`, sets the USER in the container to
  whatever the current environment variable is, or nothing if it is not set.

### `--env-file` (set a file of environment variables)

`--env-file my.env` sets all environment variables from `my.env`.

```sh
# my.env
NAME=value
...
```

### `--link name-or-id:alias`

Links the container that is being started to the container with `name-or-id`.
The link will be avaiable as `alias` inside the container. If `:alias` is left
out the alias will be the same as `name-or-id`


### `-p` and `--publish` (publish container port on host)

```sh
$ docker run -p host-port:container-port IMAGE`
```

* `-p 8080:80` will publish the container-port (80) on the host-port(8080)
* `-p 80` will publish the container-port (80) on a random host-port(8080)

### `-P` and `--publish-all` (publish all EXPOSEd ports on random ports on host)

The ports will be somewhere between 32768 and 61000 depending on your
configuration. Use `docker ps` or `docker inspect` to see what ports are used.

### `-v` and `--volume host-dir:container-dir` (mount a container-dir)

If no `host-dir` is given a new directory will be created on the host. The
`host-dir` can be located with `docker inspect`.




![logo](https://camo.githubusercontent.com/5b45fa2bae9e221a4810bb4b3999055e3929229d/687474703a2f2f7777772e6172637469717561746f722e636f6d2f6f7070656e6b616c6c6b6f642f6173736574732f696d616765732f6a61797761795f6c6f676f2e706e67)
*anders.janmyr@jayway.com*
