---
layout: post
title: Docker Cheat Sheet
---

This post i have created to bookmark so that i do not forget most important docker commands and basic syntax of Dockerfile


#### Building docker container from dockerfile in local same folder

```
docker build . 
OR 
docker build -f /path/to/a/Dockerfile .
```


#### Dockerfile Syntax
```
FROM ubuntu:16.04

# Identify the maintainer of an image
LABEL maintainer="mayank10.j@gmail.com"
ARG DEBIAN_FRONTEND=noninteractive

RUN  apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
	wget \
    git \
    curl \
    software-properties-common \
	make 

```

here `FROM ubuntu:16.04` keyword means signifies base repository to pull your image, choosing a good base docker image is very 
important. You can find thousands of base docker images on internet and choose one which is most suitable for your need.

Through `ARG DEBIAN_FRONTEND=noninteractive` flag you are declaring a environment variable `DEBIAN_FRONTEND=noninteractive` 
and applying it to system wide on your docker container. 

`RUN  apt-get update && apt-get install ... ` is command to tell Docker build system to run a command line command while 
building container. Please note docker actually builds container into layers, each layer is immutable and stacks on top of each other. So it is always good idea to combine multiple command line command into single RUN command so in order to prevent layer bloat. As each docker layer consumes additional storage space. In order to fit multiple lines of commands into single RUN command we usually put escape character `\` and immediately press `enter/newline`.
