---
layout: post
title: Docker Cheat Sheet
---

This post i have created to bookmark so that i do not forget most important docker commands and basic syntax of Dockerfile



### Dockerfile Syntax
Take a look at one of the dockerfile i had built for one of our open source project idea2life 
hosted here. 
https://github.com/keplerlab/idea2life/blob/master/ai/Dockerfile

```
FROM ubuntu:16.04

# Identify the maintainer of an image
LABEL maintainer="keplerwaasi@gmail.com"
ARG DEBIAN_FRONTEND=noninteractive

RUN  apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
	wget \
    git \
    curl \
    software-properties-common \
	make 

RUN  curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && \
    apt-get install -y nodejs \
    && \
    apt-get clean && \
rm -rf /var/lib/apt/lists/*

#Install MINICONDA
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O Miniconda.sh && \
	/bin/bash Miniconda.sh -b -p /opt/conda && \
	rm Miniconda.sh

ENV PATH /opt/conda/bin:$PATH

RUN conda install opencv flask

RUN pip install sphinx sphinx-js sphinx-rtd-theme
RUN npm install -g jsdoc


ARG PROJECT_DIR=/home/ai/
ARG HOME_DIR=/home/

RUN mkdir ${PROJECT_DIR}

# clone pyyolo and darknet
RUN mkdir ${PROJECT_DIR}/lib \
	&& \
    cd ${PROJECT_DIR}/lib \
    && \
    git clone --recursive https://github.com/keplerlab/pyyolo.git

#COMPILE DARKNET and PYYOLO
RUN cd ${PROJECT_DIR}/lib/pyyolo \
    &&  \
    make -j8  \
    &&  \
    rm -rf build \
    &&  \
    python setup.py build 


#Install PYYOLO
RUN cd ${PROJECT_DIR}/lib/pyyolo \
    &&  \
    python setup.py install    
 
COPY cfg/yolo-obj.cfg ${PROJECT_DIR}/lib/pyyolo/darknet/cfg/
COPY data/obj.names ${PROJECT_DIR}/lib/pyyolo/darknet/data/

WORKDIR ${PROJECT_DIR}
ENV FLASK_APP server/app.py
ENV FLASK_RUN_HOST 0.0.0.0
CMD ["flask","run"]
```
I will suggest to keep referring to this dockerfile to know what is being done here. 

We will go to each section of dockerfile one by one to explain what it is doing. 
First checkout few lines. 

#### FROM, ARG and RUN keyword 
```
FROM ubuntu:16.04

# Identify the maintainer of an image e.g. 
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

`FROM ubuntu:16.04` keyword means signifies base repository to pull your image, choosing a good base docker image is very 
important. You can find thousands of base docker images on internet and choose one which is most suitable for your need.

Through `ARG DEBIAN_FRONTEND=noninteractive` flag you are declaring a environment variable `DEBIAN_FRONTEND=noninteractive` 
and applying it to system wide on your docker container.

`RUN  apt-get update && apt-get install ... ` is command to tell Docker build system to run a command line command while 
building container. Please note docker actually builds container into layers, each layer is immutable and stacks on top of each other. So it is always good idea to combine multiple command line command into single RUN command so in order to prevent layer bloat. As each docker layer consumes additional storage space. In order to fit multiple lines of commands into single RUN command we usually put escape character `\` and immediately press `enter/newline`.

#### COPY keyword 
I will now skip next few lines as they seem to be self explanatory, we are installing bunch of libraries like nodejs, anaconda, 
opencv sphinx, making a  etc there.

Let's skip to a new command COPY command. See relevant lines here ... 

```
COPY cfg/yolo-obj.cfg ${PROJECT_DIR}/lib/pyyolo/darknet/cfg/
COPY data/obj.names ${PROJECT_DIR}/lib/pyyolo/darknet/data/
```
What it does? While building docker container from dockerfile it copies these files from folder where you are building
docker container to a folder inside docker container.

#### ARG keyword vs ENV keyword 
ARG and ENV both keywords can actually define environment variables, but lifetime of these environment variables is different. 
ARG based environment variables are only available during docker build state but ENV based environment variables are also available during docker build as well as run phase. So if you use want to define any environment variable that you want available inside your python programme during runtime you should define them using ENV keyword.

#### WORKDIR and CMD keyword
Take a look at last few lines of our dockerfile 
```
WORKDIR ${PROJECT_DIR}
ENV FLASK_APP server/app.py
ENV FLASK_RUN_HOST 0.0.0.0
CMD ["flask","run"]
```
so basically WORKDIR keyword defines a directory from which your docker container will start when you start the container. 
using CMD command you can define which command you want to run after your docker container is run after build process id done.

#### ADD keyword
Although it is not mentioned in my sample dockerfile, but if you want to add an host folder to an docker container you can use 
ADD keyword in Dockerfile like this: 
```
ADD . /path/inside/docker/container
```
Please note that, any changes made to this directory on the host after building the dockerfile will not show up in the container. This is because when building a container, docker compresses the directory into a .tar and uploads that context into the container permanently. 

### Building docker container

Building docker container from dockerfile in local same folder
```
docker build . 
```
using -f argument you can define path to dockerfile instead of picking docker file from your current folder. 
```
docker build -f /path/to/a/Dockerfile .
```
Using -t option you can define what should be the name of docker container after it is built from dockerfile. 
```
docker build -t dockerContainerName .
```

### RUN docker container
After you are done building a dockerfile using docker build command above you can then run docker container using command: 
```
docker run -d nodebb
```

We will now explore useful commmand line options for docker RUN.

#### MOUNT host directory to a docker container using -v :
this will mount host directory `/tmp` to a folder inside container `/container/directory`
```
docker run -v /tmp:/container/directory dockerContainerName
```
Please note any existing content of /container/directory will get hidden after this mount option. 


#### Map/Expose port from docker container using -p option: 
```
sudo docker run -p 22 -p 8000:80 dockerContainerName
```
here we are exposing 22 port of docker container to host, also we are opening 80 port of docker container to 8888 port 
of host. 
