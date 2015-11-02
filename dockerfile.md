![](http://media.bestofmicro.com/I/V/468535/gallery/docker-logo_w_195.png)
# Dockerfile Quick Reference

* Dockerfile Reference Docs [https://docs.docker.com/reference/builder/](https://docs.docker.com/reference/builder/)
* Dockerfile Best Practices [https://docs.docker.com/articles/dockerfile_best-practices/](https://docs.docker.com/articles/dockerfile_best-practices/)

A Dockerfile contains instructions for both build-time and run-time. The
instructions are marked with *BUILD*, *RUN*, or *BOTH* to indicate where they
can be used.

### docker build

```sh
$ docker build [OPTIONS] context-dir
```

Common OPTIONS are:

* `-t atag` - tag the image with `atag`
* `-f my-docker-file` - use a Dockerfile with a different name or location than
  `context-dir/Dockerfile`

`context-dir` is commonly the current directory `.`.


### `FROM: base-image` (the image this image will be based on) *BUILD*

Base my created image on the `base-image`, examples: `scratch`, `debian`,
`busybox`, `node:5-onbuild`, etc.

`base-image` may be designated with just the `image` or with `image:tag`.
If not tag is given `latest` is assumed.

### `WORKDIR container-dir` (set the working directory in the container) *BOTH*

All instructions after `WORKDIR` will be executed with `container-dir` as
working directory.

### `ADD/COPY context-path(s)... container-path` (copy a files to the image) *BUILD*

`COPY` files from the `context-path` to the `container-path`. Paths in the
container are relative to `WORKDIR`.

A `container-path` which ends with a slash is a directory and the files will be
copied into the directory.

`context-path` supports simple wildcards such as `*` and `?`. If `context-path`
refers to multiple files, `container-path` must be a directory (end with a slash)

`ADD` works like `COPY` except it can copy remote URL:s and unpack
tar-archives. (but it does not unpack remote tar-archives)


### `RUN command` (run a command at build time) *BUILD*

Run commands in the container at build time. Examples:

* `apt-get install package` - install a package.
* `rm some-files-not-needed-at-runtime` - rm some files.
* `chmod u+x /usr/local/bin/my-server` -make my-server executable.


### `CMD [command, param1]` (default command to run when running the image) *RUN*

The commands can be given as two forms:

* `CMD` ["exec", "param1", ...] - *exec form*, commands are run without any
  shell expansions, this is the preferred form.
* `CMD command...` - *shell form*, the commands are run as with `/bin/sh -c`,
  shell expansion of globs and environment variables is performed.


### `ENV NAME=VALUE` (set default environment variables) *BOTH*

`ENV` also comes in two forms:

* `ENV key value` - sets a single variable `key` to value, value is everything,
  verbatim, to the end of the line.
* `ENV key1=value1 ...` - can set multiple values, quotes, backslash, etc works
  as on the command line.

### `EXPOSE port` (expose a port) *RUN*

The port is exposed to other containers, not to the host. Use `docker run -p`
for that.

### `VOLUME /path` (path will be a volume outside the container) *RUN*

This means not handled by Docker's union file system

![logo](https://camo.githubusercontent.com/5b45fa2bae9e221a4810bb4b3999055e3929229d/687474703a2f2f7777772e6172637469717561746f722e636f6d2f6f7070656e6b616c6c6b6f642f6173736574732f696d616765732f6a61797761795f6c6f676f2e706e67)
*anders.janmyr@jayway.com*
