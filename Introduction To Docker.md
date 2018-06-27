# Introduction to docker by Stav Alfi

#### Topics

1. [Introduction](#introduction)
2. [What is a Docker?](#what-is-a-docker)
3. [Docker components](#docker-components)

![](https://www.docker.com/sites/default/files/Whale%20Logo332_5.png)

The end of this article contains the [complete terms table](#complete-terms-table).

---

### Introduction

In this article, we will define what docker is, and it's main components.

###### Prerequirements

Basic knowledge about what is a compilation, unit tests, CI (Continuous integration) and CD (continuous deployment), virtual machines, cluster, and Linux.

You can ignore any other phrase.

### What is a Docker?

In short, Docker is a software for managing CI/CD for our application. It means, we no longer need to manually install our products after each build in the development/production environment. Docker does that by letting us install multiple _tiny_ operation systems (e.g., Ubuntu) on our virtual/physical machine, and each _tiny_ operation system will contain a single web application (Why we choose to dedicate the _tiny_ OS to a single application? we will cover the reasons later).

By referring to _small_ operation system, we mean a smaller size of an operating system with fewer features than regular compared to installing an operating system on a virtual machine or our machine. How small can be an operating system in docker? So small it contains the minimal features for our web-app it can fully operate. For example, most web apps won't require the machine which they are installed on to contain some ubuntu packages as ping/git/maven and so on in production environment.

### Docker components

###### Docker engine

_Docker deamon_ is the service in our machine (which is a part of a docker application). It is a process running on our machine and is responsible for creating/removing/updating those _tiny_ operation systems.  The way we can communicate with him ( tell him what to do) is by a _docker CLI_ which we use to send commands to the docker daemon by a REST API he exposes. In short, the docker daemon is the backend, and the docker CLI is the frontend. Together they are called _docker engine_. Each operating system that you want to install small operating systems on must have a docker engine installed.

###### Docker image

When you want to install a web application on an OS, first you need an OS and then installing the requirements software for the web application you would like to finally install and then configure those software and maybe the OS you are installing. Finally, you can install your web application.

This process is too long, manual and can fail by human errors. A better way is to do it once (or describe it by commands - will be explained later) and save that OS we built with all those software and our web applications to our disk so we can use that save later again. Meaning if we use that save again, we will load the OS in the same state it was when we created that save. 

The state of an OS in a specific time is called _snapshot_ and the saved snapshot is called _docker image_. We can create multiple instances of the different/same docker image in the same OS docker engine is installed. 

Docker let us create an image from an image. Meaning, we can run an image and change as we like anything in that _tiny_ OS and then create a new image from the current snapshot of that _tiny_ OS. We just run. 

###### Docker repository

Each image is dedicated to a single web application. It is possible that you would like to create multiple images containing the same web application with different settings or different versions of software your web application need or remove/add/change settings of the OS for different use cases. All in all, you may find your self in a situation you have multiple images for the same web application. Docker let us use _tags_ to differentiate multiple different images which are dedicated to the same web application. A tag is an additional string which will be presented alongside the name of the image. 

Multiple Images with the same name but contain different tags are called _docker repository_.

###### Docker registery

Is it possible and highly recommended to share images between users. There are _places_ where one can upload images, manage them and download others. This kind of place is called a _docker registery_.  Docker registry is a service which manages docker repositories.

Today there is an image for most of the web applications in multiple versions. It means multiple users created an image dedicated to the same web application but with some changes in the web applications or the OS that web application is installed on. It means different users manage different docker repositories for the same image. 

To download and run images on our OS, you can enter one of many docker registries out there (e.g., https://hub.docker.com)

Search an image by name, download and run it! (How? Will be covered in the different article).

###### Docker container

When we "running" an image, we are creating some "virtual machine" which load that image. An image is a snapshot of an operating system, so all we need to do is to run that image somewhere. _Docker container_ is the tool which runs an image. Containers share some similarities with virtual machines. They are running OS, and you can connect to its shell and run any script you want.

Docker containers are fast. They communicate with the host OS kernel. We can configure multiple containers to be visible to one another while still invisible from the outside world. We choose which ports we open for communicating containers in the same host the docker engine is installed (It is the one who manages images and containers) and which ports will be visible to the outside world. We can relate OS ports to specific container ports. It means when we try to reach `localhost:X`, we are reaching container1_ip:X.

We can also restrict the number of resources which will be used by a container from the host OS. But this will be discussed in a future tutorial.

The best practice is to load one web application in each container. From what we described, we can load the same image multiple times to have a cluster for example. (a Cluster is a group of physical/ virtual machines that operate as one).

Check swarm for more information about docker cluster management.



###### Dockerfile

Another way to create an image is by describing what we want it to look like using individual commands/instructions in a file called _dockerfile_. Then we create an image from that docker file. 

The `dockerfile` is divided into 3 parts: 

1. A _base image_. It can be a simple ubuntu image or an IIS image which on top of that we will describe (by dockerfile instructions) what we want to change in it. You can use any image you want. It is a requirement to tell what the base image you will use is explicit. This instruction is called ```FROM <image-name:tag>```.
2. Additional and optional instructions that responsible for changing the state of the base image.
3. Optional instruction responsible for running a web application this image is dedicated to. This command can be overridden when we "start and run" the image. It will run as soon as the image is "running". This instruction is called ```CMD command param1 param2``` (It is not the windows CMD).

The `dockerfile` is transformed into an image using a temporary container which runs the `dockerfile` and saves the container's snapshot to an image.


![](https://i.imgur.com/U4GnCD8.jpg)


###### Docker service

There is a need to control multiple containers which all run the same image (for cluster management). A _docker service_ is a couple of containers running the same image with a defined state. Docker let us describe the desired state of service: The amount of _replicas_ (multiple containers running the same image), minimum resources that a container must have for running a replica, what is the image we are trying to deploy and so on. The description will be written in a yml file written in `yaml` language:

```yaml
services:
  web:
    image: stavalfi/myimage1:latest
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
```

Who is deploying and managing (ensuring that the state of the service will be as described in the `yml` file) the service/s? The `docker-engine`. More specifically, an existing and running _swarm_. Before we talk about swarms, we need to understand how to describe an application which contains multiple services using yaml.

###### Docker stack

Docker service is described using yml file. We can describe multiple services in the same file and this is called _docker stack_:

```yaml
services:
  web:
    image: stavalfi/myimage1:latest
    deploy:
      replicas: 5
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - /home/docker/data:/data
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```

Here we are describing multiple services' desired state: _web_, _visualizer_, _redis_ (those are only the name we chose to refer to those services).

###### Docker swarm

> **Definition 1.11.** ___Docker container orchestration___  is a service which able to manage containers in different virtual/ physical machines. It is responsible for running/stopping/suspend containers and also preserves a state that is defined for every service in a stack it manages.

> **Definition 1.12.** ___Docker swarm___  is a docker container orchestration service from docker company.

In simple words, docker swarm is a stack manager which ensure that the state described by the yml file will preserve. Swarm consists of multiple nodes (one or more virtual/ physical machines). Each node can be a _worker node_ or _manager node_. The creator of the swarm is a manager and also the _leader node_. He can promote or provoke nodes to worker/manager degree. 

> **Definition 1.13.** ___Task___  is a request from a worker node to run container with a specific image. The task is in a _running_ state only after the container is in a _running_ state. In case of any problem with the container, it states will also change and will cause the task to change its state and send it's new state to one of the managers to examine the problem.

To examine a docker task let's describe an example where a container is down, then it's task is terminated and as a result, one of the managers will automatically create a new task with the same image in one of the nodes in the swarms which can be assigned with new tasks. The managers or the leader can assign the task to a node in any type, even them selfs (but they will prefer to assign a task to a worker node).

> **Definition 1.14.** ___Worker node___  is a node who does not have any permission to modify or examine the state of the swarm nor see the nodes in it. It has a docker engine installed, and it checks with one of the managers for new tasks or gets requests from the swarm's load balancer to a specific container in it.

> **Definition 1.15.** ___Manager node___  is a worker node which promoted by the leader to a manager degree. It can change the swarm state; stop/run/rm services in the swarm. The manager assigns tasks to nodes in the swarm (Including himself if this node is configured to run tasks) to preserve the state of a service in a swarm. Manager node checks the health of worker nodes and the status of all the tasks he assigned to examine any change.

> **Definition 1.16.** ___Leader node___  is the node which created the swarm. He gave full privilege to control the swarm state same as a manager node. It also can Promote worker nodes to manager nodes or vice versa. 

###### More about swarm services

Services are only running inside a swarm. There are two options to deploy a service in a swarm:

1. _Replicated services model_ - We explicitly specify how much replicas will be running inside a swarm. It means that if he wants 5 replicas for image `stavalfi/projecty:latest`,  the swarm managers will try to create 5 tasks in the swarm. They will try to balance the number of tasks created in each node, depending on how much running tasks it run at the moment. 
2. _Global services model_ - Each service will be deployed in all the swarm's nodes. This option is suitable for deploying a monitoring/anti-virus/.. services.

For every service, we can specify conditions which every node want to run a task need to fulfill them. For example, we can deploy service only in manager nodes. Also, we can specify how many minimum and maximum resources will be used by each service's container. It means that if a node does not have the minimum resources for running that container as we explicitly specify, then that task won't send to that node. If a no node can fulfill all the conditions for a service, all its tasks won't start; the task will be in a _pending_ state.

The following image is from [docker docs](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#services-tasks-and-containers):

![](https://docs.docker.com/engine/swarm/images/service-lifecycle.png)



---

### Complete terms table

> **Definition 1.1.** ___Docker deamon___ it is a process running on our machine and is responsible for managing containers and images.

> **Definition 1.2.** ___Docker CLI___ is a client tool we can use from the command line to send commands to the REST API of the docker daemon in the same operating system to tell him what we want him to do.

> **Definition 1.3.** ___Docker engine___ is the docker deamon and the docker CLI.

> **Definition 1.4.** The state of an OS in a specific time is called _snapshot_ and the saved snapshot is called ___docker image___.

> **Definition 1.5.** ___Docker tag___ is describing the version of an image (Not only numeric version).

> **Definition 1.6.** Multiple Images with the same name but contain different tags are called ___docker repository___.

> **Definition 1.7.** ___Docker registery___ is a service which provides a way to manage our docker repositories and let us download different docker images from different users.

> **Definition 1.8.** ___dockerfile___ is  a file described by special instractions/commands to define an image.

> **Definition 1.9.** ___Docker service___ is a couple of containers running the same image with a defined state.

> **Definition 1.10.** ___Docker stack___ is a couple of docker services.

> **Definition 1.11.** ___Docker container orchestration___  is a service which able to manage containers in different virtual/physical machines. It is responsible for running/stopping/suspend containers and also preserves a state that is defined for every service in a stack it manages.

> **Definition 1.12.** ___Docker swarm___  is a docker container orchestration service from docker company.

> **Definition 1.13.** ___Task___  is a request from a worker node to run container with a specific image. After the container is up, the task is running, and any change in the container will cause the task to change its state and send it's new state to one of the managers to examine the problem.

> **Definition 1.14.** ___Worker node___  is a node who does not have any permission to modify or examine the state of the swarm nor see the nodes in it. It has a docker engine installed, and it checks with one of the managers for new tasks or gets requests from the swarm's load balancer to a specific container in it.

> **Definition 1.15.** ___Manager node___  is a worker node which promoted by the leader to a manager degree. It can change the swarm state; stop/run/rm services in the swarm. The manager assigns tasks to nodes in the swarm (Including himself if this node is configured to run tasks) to preserve the state of a service in a swarm. Manager node checks the health of worker nodes and the status of all the tasks he assigned to examine any change.

> **Definition 1.16.** ___Leader node___  is the node which created the swarm. He gave full privilege to control the swarm state same as a manager node. It also can Promote worker nodes to manager nodes or vice versa. 


------------------

### Legal

© Stav Alfi, 2017. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Introduction to docker by Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/eea616c2aaf5299a84c718a77cfe8668.