# Connecting Docker Images Part one

https://deis.com/blog/2016/connecting-docker-containers-1/

We have three ubuntu images, one with Apache2 and the other two with multichain soft.

### Exposing ports

First, let’s see how we can run a server container so that it exposes port 80 (HTTP) to other containers.

Use "expose" it's where other containers can reach

from server image:
```
$ sudo docker run -itd --expose=80 --name=server1 server_img /bin/bash
$ docker attach server1
```
inside server1:
```
$ /etc/init.d/apache2 start

```

from client image:
```
$ sudo docker run -itd --name=client1 multichain_img /bin/bash
$ sudo docker attach client1

```
inside client1:
```
$ curl ip_server1

```

### Port binding

Expose our HTTP server to the host network.
To expose the Apache HTTP server to the host network, we need to bind port 80 on the server container to port 8080 on the host machine.

```
$ sudo docker run -itd -p 8080:80 --name=server2 server_img /bin/bash

```

Go to localhost:8080 in your host machine


### Linking containers

When you link one container to another, Docker will make some information about the linked container available via environment variables.

Also the file /etc/hosts in client3 appears the server3

Start the server:
```
$ sudo docker run -itd --name=server3 server_img /bin/bash

```
Start client container and link it to the server:

```
$ sudo docker run -itd --link server3 --name=client3 client_img /bin/bash

```

Attach to the client3 and check the enviornment variables:
```
$ sudo docker attach client3
$ env | grep SERVER3

```
