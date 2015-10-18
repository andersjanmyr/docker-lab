# Docker Run Quick Reference

[Docker Run Reference Docs](https://docs.docker.com/reference/run)

### docker run

```
$ docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]

`IMAGE` is the image name, with or without a namespace. The only images without
a namespace are official images such as: *debian, node, redis*, etc. All other
images have namespaces such as: *andersjanmyr/counter* or *ansible/centos7-ansible*.

`TAG` is any tag that exists for the given name. *latest* is used if not tag is
given.

`COMMAND` is any command that the image supports, it defaults to the `CMD`
given in the Dockerfile.
```

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

### `--rm` (remove the container when it stops)

Commonly used when running containers as tools. 




