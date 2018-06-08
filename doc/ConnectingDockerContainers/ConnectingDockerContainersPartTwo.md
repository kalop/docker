# Connecting Docker Containers Part Two
https://deis.com/blog/2016/connecting-docker-containers-2/

In this post, we’ll look at a more advanced, and up-to-date use of the bridge network driver.
We’ll also look at using the overlay network driver for connecting Docker containers across multiple hosts.

### USer defined Networks

Two containers to communicate, all that is required is to place them in the same network or sub-network

Let’s create a network :
```
$ sudo docker network create backend

```
* Here we can see the backend network has been created using the default bridge driver.

Run a server container from the server_img image and put it on the backend network using the --net option:

```
$ sudo docker run -itd --net=backend --name=server server_img /bin/bash

```
Attach the server and run the apache http server:
```
$ sudo docker attach server
$ /etc/init.d/apache2 start

```
At this point, any container on the backend network will be able to access our Apache HTTP server.

### Multi-host networking docker-machine
