## Need For Docker
* Software stack includes lots of services that need to be able to communicate each other. All these services should be compatible with
the existent os kernel. There have been times when certain version of these services were not compatible with the OS. In this case
we look for compatible OS.

* Secondly libraries and dependencies need to be compatible in the services. Everytime services can change and we need to be sure
about the compatibility overall.

* Decrease the time we waste to building environment like dev/test/prod.

* With docker, we can run each component in a separate container with its own dependencies and its own libraries.

## What are containers?
* Containers are completely isolated environments. They can hace their own processes or services with their own network interfaces 
like virtual machines except they all share the same OS kernel.

## VM vs Container
* Containers are lightweight
* Vms have limited performance
* All containers share the host OS
* OS virtualization actually in containers
* Startup time is small in containers
* Allocates less memory in containers

## Container vs Image
* And image is a packaged template to run a container. Can contain one or more services.
* Containers are running instances of images.

# Docker Run
* Every docker container gets an ip assigned to it. This is internal and only available on the docker host.
* To be able to access docker container externally, we need port mapping.
```
docker run -p 80:5000 redis
```
* This command maps traffic comes from port 80 to 5000 port in docker host. We can also create more instances on the same container
via another ports.
* All data within the container lives in containers file system. So if we stop and remove the container data will be lost.
We can persist this data on the docker host using the volume mapping.
```
docker run -v /opt/datadir:/var/lib/mysql mysql
```
* This commands mount the var/lib/mysql folder to the /opt/datadir folder which exist in the docker host. 

# Docker Images
## Containerizing an App
To be able to containerize a docker file follow these steps
* Provide base os
* Update apt repo
* Install dependencies using apt
* Install python dependencies using pip
* Copy source code to /opt folder
* Run the web server

Then build the file on locally:
```
docker build . -f Dockerfile -t <image-name>
```
or
```
docker build -t <image-name> .
```

Then push image to the registry
```
docker push <image-name>
```

Docker file consists of instructions and arguments.
Every docker image should be base on the another image or the OS.
To summarize:
* Start from a base OS or another image.
* Install all dependencies
* Copy source code to image
* Specify enrypoint

This is called layered architecture.
Each layer is based upen previous layer.
All the layered are cached. 
So when you change something on the file, it builds fast because only the changed layered will be affected.

## Environment Variables
We can pass an environment variable to the source code lives in container.
```
docker run -e ENV-VAR-NAME=value <image-name>
```

## Command vs Entrypoint
When we run an ubuntu image, it runs for a little then disappear. Its because containers are designed to provide a service
not OS.
<br> 
This behaviour is specified in the Dockerfile with CMD instruction.
<br>
If we look at the ubuntu image, we can look the 'bash' option in CMD instruction. 
By default docker doesn't attach a terminal to the container, and bash doesn't find input then exists.
<br> 
How can we override this setting to behave differently
Thats an example:
```
docker run ubuntu sleep 5
```

CMD structure is like this:
<br>
CMD command param1
<br>
CMD["command", "param1"]

ENTRYPOINT[command]

* Both CMD and ENTRYPOINT instructions define what command gets executed when running a container. There are few rules that describe their co-operation.

Dockerfile should specify at least one of CMD or ENTRYPOINT commands.
<br>
ENTRYPOINT should be defined when using the container as an executable.
<br>
CMD should be used as a way of defining default arguments for an ENTRYPOINT command or for executing an ad-hoc command in a container.
<br>
CMD will be overridden when running the container with alternative arguments.
[see](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)

# Docker Compose
### Links
One service can depend on another. For ex: a webapp depends on a redis service.
In this situation we need to create a link.

<br>
An example of docker compose file like this:

```
redis:
    image:redis
db: 
    image:postgre
    ports: 
        - 5000:80
    links:
        - redis
```

Some components that we need to use in docker compose: services, images, ports, links, networks

# Docker Engine
Consists of three components:
* Docker CLI
* REST API
* Docker Daemon

Daemon is a process manages objects such as images, containers, volumes, networks.
<br>
CLI is the command line utility and it can interact with the daemon via rest api.
<br>
CLI doesnt need to be in the same machine with the daemon and rest api.
Just use this command to connect it via remotely:
```
docker -H=remote-docker-engine:2375
```
## Containerization
Docker uses namespaces to isolate the workspace. Processes, network, mount, unix timesharing, interprocess
<br>
This namespaces isolated between containers.

# Docker Storage
When we install docker on a machine, it create a file system like this:
```
/var/lib/docker
    -aufs
    -containers
    -image
    -volumes
```

# Docker Networking
* Docker creates three networks automatically. Bridge, none, host 
* Bridge is the default network a container gets attached to

### Bridge
* Private internal network, ip ranges 172.17.x.x
### Host
* No need to define port mapping. Because when you create a container, it is accessible externally.
###
* Containers are not attached to any network and doesn't have access to external network or other containers.
Run in an isolated network.

## Embedded DNS
* When an app. wants to try a container, it can define container id, not ip address. Because address can change when reboot happens.
Docker has embedded dns resolver and can handle.
