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
