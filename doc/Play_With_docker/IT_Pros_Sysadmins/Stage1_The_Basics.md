# Play with Docker Classroom
https://training.play-with-docker.com/
## Getting Started Walk-through for IT Pros and System Administrators

### Stage 1 - The Basics
https://training.play-with-docker.com/ops-stage1/

#### What will we learn?
* Get familiar with core concepts
* Understand adopt Docker
* How Docker can help

#### Content:
* [1.Your First Linux Containers](#1-your-first-linux-containers)
* [2.Customizing Docker Images](#2-customizing-docker-images)
* [3.Deploy and Managing Multiple Containers](#3-deploy-and-managing-multiple-containers)

#### 1. Your First Linux Containers
https://training.play-with-docker.com/ops-s1-hello/

Concepts:
* Docker engine
* Containers & images
* Image registries and Docker Store (AKA Docker Hub)
* Container isolation

##### 1.0 Running your first Container
```
$ docker container run -->
Tries to run a container, if it can't find it it pulls the image
```


##### 1.1 Docker Images
```
$ docker image pull alpine --> alpine is alightweight Linux
$ docker image ls --> list of images
$ docker container run alpine ls -l --> start container, run the command and exit
$ docker container run alpine /bin/sh --> a shell is opened and exit

```
To get an interactive shell: -it
```
 docker container run -it alpine /bin/sh

```
To show all containers running:
```
$ docker container ls
```
To show all containers running and not: -a
```
$ docker container ls -a
```
##### 1.2 Container isolation
We had run each of our commands above in a **separate container instance** --> **Isolation** each command uses the same image, but a isolated **container**. A container has no way to interact with other

Open a ash shell
```
$ docker container run -it alpine /bin/ash

```
ash shell waits for a command, then we restart the container:
```
$ docker container ls -a
$ docker container start <id>

```
This time the instance is still runnig, ash is waiting a command

To send acommand:
```
$ docker container exec <id> ls

```

##### 1.3 Terminology


* **Images** - The file system and configuration of our application which are used to create containers. To find out more about a Docker image, run docker image inspect alpine. In the demo above, you used the docker image pull command to download the alpine image. When you executed the command docker container run hello-world, it also did a docker image pull behind the scenes to download the hello-world image.
* **Containers** - Running instances of Docker images — containers run the actual applications. A container includes an application and all of its dependencies. It shares the kernel with other containers, and runs as an isolated process in user space on the host OS. You created a container using docker run which you did using the alpine image that you downloaded. A list of running containers can be seen using the docker container ls command.
* **Docker daemon** - The background service running on the host that manages building, running and distributing Docker containers.
* **Docker client** - The command line tool that allows the user to interact with the Docker daemon.
* **Docker Store** - Store is, among other things, a registry of Docker images. You can think of the registry as a directory of all available Docker images. You’ll be using this later in this tutorial.

#### 2. Customizing Docker Images
https://training.play-with-docker.com/ops-s1-images/
Concepts:
* image creation with simply **commit** one of our continer instances as an image
* USe Dockerfile to create images

##### 2.0 Image creation from a containers

First modify and install programs in a container. For example install a useful program in an Ubuntu container

```
$ docker container ls -a --> to pick the id
$ docker container diff <id> --> lists all the modifications we did
```
To create the container we have to commit it
```
$ docker container commit <id> --> the image is created without name and tag

```

Tagging the image
```
$ docker image tag <IMAGE_ID> ourtest

```
##### 2.1 Image creation using Dockerfile

Example:
* Create an index.js (node.js program that we want to run in our container)
* Create a Dockerfile
    ```
    FROM alpine  --> base image to pull FROM
    RUN apk update && apk add nodejs  -->  installs the Node.js server
    COPY . /app  --> COPY files from our working directory in to the container
    WORKDIR /app  -->  specify the WORKDIR
    CMD ["node","index.js"]  --> gave our container a command (CMD) to run

    ```
    Build the image:
    ```
    $ docker image build -t hello:v0.1 .
    $ docker container run hello:v0.1
    ```

##### 2.2 Image layers
A container is bulit with layers, it means that a change maybe only affect one layer.
```
$ docker image history <image ID>
# modify index.js
$ docker image build -t hello:v0.2 . --> in some steps it use cache
```

##### 2.3 Image Inspection
We can see the layers inside the image
```
$ docker image inspect <image ID>
$ docker image inspect --format "{{ json .RootFS.Layers }}" <image ID>

```

##### 2.4 Terminology
* **Layers** - A Docker image is built up from a series of layers. Each layer represents an instruction in the image’s Dockerfile. Each layer except the last one is read-only.
* **Dockerfile** - A text file that contains all the commands, in order, needed to build a given image. The Dockerfile reference page lists the various commands and format details for Dockerfiles.
* **Volumes** - A special Docker container layer that allows data to persist and be shared separately from the container itself. Think of volumes as a way to abstract and manage your persistent data separately from the application itself.

#### 3. Deploy and Managing Multiple Containers:
https://training.play-with-docker.com/ops-s1-swarm-intro/

Concepts:
* Compose: To control multiple containers on a single system with a configuration file
* Swarm Mode: To run many Docker engines and coordinate operations across all of them

##### 3.0 Initialize your swarm

Usually there are three manager nodes and many workers. Three managers is the minimum to have a true high-availability cluster

Init it:
```
$ docker swarm init --advertise-addr $(hostname -i)

# To add more managers to this example:
$ docker swarm join-token manager

# To add more workers
$ docker swarm join --token SWMTKN-1-0dg418ldbplswsdbxf6ug7e67nr5ifqyq66yy7drdb6ke288cg-9l4nptblzy3j4gxxb7txwv6lp 192.168.0.33:2377

```
This swarm manager is listening on the IP returned by hostname -i

Add a worker in other machine

##### 3.1 Show Swarm Members

```
docker node ls
```

##### 3.2 Example, Clone the Voting app
```
git clone https://github.com/docker/example-voting-app
cd example-voting-app

```

##### 3.3 Deploy a stack

The file *docker-stack.yml* defines the stack
```
$ docker stack deploy --compose-file=docker-stack.yml voting_stack
$ docker stack ls
```
Show the services:
```
$ docker stack services voting_stack

```
List the task of a specific service (replicas)
```
$ docker service ps voting_stack_vote

```
##### 3.4 Scaling an applications
How we can tell our app to **add more replicas** of our vote service? In production --> automate with Docker's alpine

Manually:
```
$ docker service scale voting_stack_vote=5

```

##### 3.5 Terminology
* **Stack**: Group of inter related services & dependencies
* **Service**: A stack component
* **Tasks**: Atomic unit of a services
