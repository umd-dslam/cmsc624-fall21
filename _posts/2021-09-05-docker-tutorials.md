---
layout: distill
title: "Docker Tutorial"
description: Introducing why Docker is useful, and an overview of the main commands of Docker.
date: 2021-09-05

lecturers:
  - name: Daniel Abadi
    url: "https://www.cs.umd.edu/~abadi/"

authors:
  - name: Michael Marsh # author's full name
    url: "https://scholar.google.com/citations?hl=en&user=UKt7iIYAAAAJ&view_op=list_works&sortby=pubdate"  # optional URL to the author's homepage
  - name: Gang Liao  # author's full name
    url: "https://gangliao.me/"  # optional URL to the author's homepage

editors:
  - name: Gang Liao # editor's full name
    url: "https://gangliao.me/"  # optional URL to the editor's homepage

abstract: >
  If you're using Windows or Mac OS (M1 chips),  you can use docker for 
  all assignments. It's worth taking some time to familiarize
  ourselves with it, especially since you're unlikely to be familiar
  with it.
---


### Docker Images

What is docker? You can think of it like a lightweight VM. It's
really considerably different, because it uses the host processor,
memory, network stack, etc., without creating virtual hardware. We
can throw around terms like user-level filesystems, process groups,
and network namespaces, but the important part is that you can run
a self-contained guest Linux OS within another host Linux OS, with
applications and all of their dependencies.  The guest can only see
the resources given to it by the host, so it provides some (minimal)
level of security. It also means we can start a process from a
known-clean state, so we have repeatability.

Let's start with the concept of an **image**. This is the self-contained
guest Linux OS, which is configured to automatically run some process
when it starts.  Nothing is running in it -- you can think of it
like a hard drive.

The easiest way to get an image is to **pull** it from a **registry**. Docker
has a default registry built in. Our VM is running Ubuntu 16.04, and it turns
out there's an image available with this OS on it! Here's the command to run:

```shell
docker pull ubuntu:16.04
```

Let's go through this command. "docker" is, of course, the utility
we're using. The "pull" command tells us that we want to get something
from a registry. In this case, we're getting the "ubuntu" image
from the default registry. If we just left it at this, we'd get
**all** of the ubuntu variants. Instead, we add `:16.04`. That tells
docker we only want one image, and it's the one with the **tag**
`16.04`.

When the command completes, try running

```shell
docker images
```

You should see something like:

```shell
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               2a4cca5ac898        28 hours ago        111MB
```

Most of this should be fairly self-explanatory. The image ID is
another hexadecimal number, like with git, but it's clearly not a
SHA-1 hash. It really doesn't matter what it is, other than a unique
identifier for this image.

We can do a few things with this image, aside from running it. Try
the following:

```shell
docker tag ubuntu:16.04 my_ubuntu
docker images
```

Note that we now see the same image ID twice, but with different
names. By default, a repository (the tagless part of an image name)
is tagged as "latest" if you don't specify one. Let's try specifying
a tag, though:

```shell
docker tag ubuntu:16.04 foo:bar
docker images
```

The results should not be surprising.

We can quickly build up a lot of images we don't want anymore, so
it's good to know how to clean these up. Let's get rid of our new
tagged images:

```shell
docker rmi my_ubuntu:latest foo:bar
docker images
```

A common problem is that we'll end up reusing an old tag, leaving
an image with no `repository:tag` name. These show up as "<none>:<none>".
We can get rid of all of these with the following bash one-liner:

```shell
docker images -a | grep none | awk '{print $3}' | xargs docker rmi
```

For the curious, feel free to read the man pages for `awk` and `xargs`.
This is not going to be essential information for this course,
though.

The commands here are largely from an older version of docker. Now they're
aliases to new-style commands. Here's the mapping:

```shell
|  Old Command  |    New Command    |
| ------------- | ----------------- |
| docker images | docker image list |
| docker pull   | docker image pull |
| docker rmi    | docker image rm   |
| docker tag    | docker image tag  |
```


### Running an Image in a Container

Images are all fine and good, but we actually want to use docker to **do**
something, which means we have to run these images. An image runs in a
**container**. The container has system resources allocated to it, and runs
a program or programs that exist in the image. A container runs a single
image, but an image may be running in multiple containers.

Containers can also be started with various options, such as elevated
privileges, mounted volumes, environment variables, and so on. The most
basic invocation is

```shell
docker run ubuntu:16.04
```

If you run this, you'll find that it pauses for a second or so, and then
returns to the command line. If you want to see running containers, run

```shell
docker ps
```

You see headings, but probably no actual containers. Now, try

```shell
docker ps -a
```

Now we have something! Here's an example of what you might see:

```shell
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
1b937126d5bc        ubuntu:16.04        "/bin/bash"         About a minute ago   Exited (0) About a minute ago                       upbeat_archimedes
```

Let's parse this out:

- The container ID is a unique ID, like the image ID we saw before
- The image should be self-explanatory
- The command is what the container ran. In this case, it's just bash
- The created time is when the container was started
- The status tells us that this container exited, and is no longer running
- We have no ports bound, but if we did these would map from local
network ports to network ports on the container
- The names are symbolic names used to refer to this container, and are
synonyms for the container ID

By default, names are assigned randomly according to the pattern
`<adjective>_<scientist>`.

We can assign a name to the container, which is often useful:

```shell
docker run --name=bash_test ubuntu:16.04
```

This will behave similarly to the previous command, but if we run

```shell
docker ps -a
```

We'll now see our container named "bash_test" along with whatever random name
our first container was assigned.

Usually, an image is defined to do something useful when run non-interactively.
We can get interactive access to the container, though, as follows:

```shell
docker run -ti ubuntu:16.04
```

We've passed two new options to docker run. The `-t` option allocates a
pseudo-TTY, and the `-i` option makes the container interactive. You should
now have a shell on the container running as root! If you run `docker ps` in
another terminal, you will see that the container status is
"Up <length of time>".

When you're done playing around in this shell, exit to stop the container.

At this point, you probably want to get rid of these stopped containers. Run:

```shell
docker rm bash_test
docker ps -a
```

You'll still have the two randomly-named containers, but the one named
"bash_test" should no longer be present. Remove the other two, as well.

We don't have to run the configured program in a container; we can run any
command that's present on the image. Let's see this in action:

```shell
docker run ubuntu:16.04 /bin/date
```

That should print the date in the container. It's probably in UTC, while running
`/bin/date` on your VM should print the date in Eastern US time (EST or EDT). You
can also specify options:

```shell
docker run ubuntu:16.04 ls /var
```

Another very useful option is `--rm`, which will get rid of the container once
it stops:

```shell
docker run --rm --name="rm_test" ubuntu:16.04 ls /var
```

We've once again been using old-style commands, which are aliases:

```shell
| Old Command |     New Command      |
| ----------- | -------------------- |
| docker run  | docker container run |
| docker ps   | docker container ls  |
```

### Stopping a Running Container

A container might become unresponsive, or it might be a long-running service
that you want to terminate. You can do this with either of the following:

```shell
docker kill <container>
docker stop <container>
```

`stop` is more graceful, trying SIGTERM first, and then SIGKILL. `kill` sends
SIGKILL by default, but this can be overridden on the command line.

```shell
| Old Command |      New Command      |
| ----------- | --------------------- |
| docker kill | docker container kill |
| docker stop | docker container stop |
```


### Removing Stopped Containers

As with images, you'll tend to accumulate lots of stopped containers, unless
you've run them all with the "--rm" option. Fortunately, we can get rid of
these with

```shell
docker rm <container>
```

which is now an alias for

```shell
docker container rm <container>
```


### Other Options for Running Containers

Here are some useful options you might want to use:

```shell
| Option |    Argument     |                  Effect                   |
| ------ | --------------- | ----------------------------------------- |
| --rm   |                 | removes container after exit              |
| -ti    |                 | run interactively with a pTTY             |
| -e     |     <vars>      | set environment variables                 |
| -h     |   <hostname>    | set the container's hostname              |
| -p     | <hport>:<cport> | map host's <hport> to container's <cport> |
| -v     |  <hdir>:<cdir>  | mount host's <hdir> on <cdir>             |
```

### Executing Commands in a Running Container

Sometimes you need to examine what's going on inside a container. That's
where the **exec** command can come in handy. It's a lot like **run** but for
a container, rather than an image. Here's a common thing you might want to
do:

```shell
docker run --name=svc_instance my_service:latest
docker exec -ti svc_instance /bin/bash
```

What this does is to first start a container using the latest version of
the image `my_service`, and name the container `svc_instance`, and then to
execute an interactive bash shell on that container. You don't have to exec an
interactive command, though. There may be times when you want to run something
like:

```shell
docker exec svc_instance touch /var/cache/magic_file
```

in order to change the behavior of a running process. As with the other commands
we've looked at, `docker exec` is now an alias for `docker container exec`.


### Getting Process Output

Many processes send their output to STDOUT or STDERR. Since there's no TTY
available to the process in a container, this output would generally be lost.
Docker saves this output for you, however, and you can retrieve these by
running

```shell
docker logs <container>
docker container logs <container>
```

The first command is now an alias for the second command. There are a number
of options, such as `--since` to limit the timeframe of the logs returned,
`-f` to continue to follow the logs rather than just dumping their current
contents and exiting, and `-t` to show timestamps at the beginnings of lines.
