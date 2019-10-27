# Docker Management

1. [Basic Commands](#1-basic-commands)
2. [Docker Networking](#2-docker-networking)
  - [Exposing Ports](#exposing-ports)
  - [Linking Directly](#linking-directly)
  - [Private Network](#private-network)
3. [Container Volumes](#container-volumes)
  - [Sharing Data with the Host](#sharing-data-with-the-host)
  - [Sharing Data Between Containers](#sharing-data-between-containers)
4. [Transferring Docker Containers-Images as a File](#4-transferring-docker-containers-images-as-a-file)
5. [Uploading an Image to Dockerhub](#5-uploading-an-image-to-dockerhub)

### 1. Basic Commands

Command | Description
------- | -----------
`$ docker images` | Lists images installed on local system.
`$ docker ps` | Lists running containers.
`$ docker run -ti ubuntu bash` | Starts a container running bash from ubuntu image. An interactive bash terminal is connected in foreground.
`$ docker run -d -ti ubuntu bash` | -d: starts the container in background. To bring the container foreground: `docker attach CONTAINER-NAME`
`$ docker ps -a` | Lists all container including exited ones.
`$ docker ps -l` | Shows the last exited container.
`$ docker commit sleepy-einstein my-ubuntu` | Converts the container named "sleepy-einstein" to a image named "my-ubuntu".
`$ docker exec -ti sleepy-einstein bash` | Runs bash (or any other) process interactively in the container sleepy-einstein.
`$ docker run --name example -d ubuntu bash -c "cat /etc/passwd"` |  Runs the bash command (cat /etc/passwd) in the container named "example".
`$ docker logs example` | Prints the output of the command in the created container above, that is, content of _/etc/passswd_.
`$ docker rm example` | Deletes the container named _example_.
`$ docker rmi my-ubuntu` | Deletes the image named _my-ubuntu_.

> To detach container console without exiting, press **ctrl+p, then q**.
----
### 2. Docker Networking

#### Exposing Ports
```
$ docker run -ti --rm -p 2222:22 ubuntu bash
```
> The first number 2222 is for host computer that is running docker. The second number 22 is for the container listening port.

> The command above binds the port 2222 to all interfaces on host. In order to bind just specified interface, let's say localhost:

```
$ docker run -p 127.0.0.1:2222:22/tcp eg_sshd
```

##### _Ssh enabled container example:_
```
$ docker run -d -P --name my-ssh-server eg_sshd  # -P exposes all ports
$ docker exec -ti my-ssh-server passwd root
$ docker port my-ssh-server  # see port mapping
$ docker inspect my-ssh-server  # see ip configuration
```
_Or ssh key login without password is possible:_
```
$ docker exec -ti my-ssh-server passwd -d root
$ docker cp host_public_key_location my-ssh-server:/root/.ssh/authorized_keys
```
> The image named **eg_sshd** is built from a Dockerfile located in [Docker Documentation](https://docs.docker.com/engine/examples/running_ssh_service/):

```dockerfile
FROM ubuntu:16.04

RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:THEPASSWORDYOUCREATED' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```
The dockerfile lines above should be saved as _Dockerfile_. And to build an image, following command is entered within the same directory:
```
$ docker build -t eg_sshd .
```
#
#### Linking Directly

```
$ docker run --rm -ti --name server ubuntu bash
$ docker run --rm --ti --link server --name client ubuntu bash
```
If you check the content of /etc/hosts in the container ***server***, there will be seen an entry for the container ***client***.
> These types of links break when the containers exit.

#
#### Private Network

- This is the way to make links not break.
- This type of network has built in nameserver that fix the links.
- The private network must be created in advance.

```
$ docker network create example-net
$ docker run --rm -ti --net=example-net --name server ubuntu bash
$ docker run --rm -ti --link server --net=example-net --name client ubuntu bash
```

----
### 3. Container Volumes

```
Virtual Disks
           └──> Persistent
           └──> Ephemeral
```

#### Sharing Data with the Host
- Persistent
- Not part of images

```
$ mkdir ./example
$ docker run -ti -v ~/local_host_dir:/container-dir ubuntu bash
```
#### Sharing Data Between Containers
- Ephemeral
- Exists as long as they are being used
- Can be shared between containers

```
/* First start a container with volume to be shared. */
$ docker run -ti --name first-container -v /container-vol-dir ubuntu bash
/* Then start another container to reach that volume of the former one. */
$ docker run -ti --volumes-from first-container ubuntu bash

```

----
### 4. Transferring Docker Containers-Images as a File
Containers can be converted to images, be saved as a file,be transferred to somewhere else and be loaded there.

The steps to implement those:
1. Get the number or name of docker containers.
2. Commit those containers as images.
3. Save it to a tar file.
4. Load it in another system running docker service.
```
$ docker ps -a
CONTAINER ID  IMAGE        COMMAND     CREATED       STATUS                    PORTS NAMES
ecac70d6c2aa  ubuntu       "/bin/bash" 8 weeks ago   Exited (0) 20 hours ago         upbeat_curran
f08c0492d537  hello-world  "/hello"    7 months ago  Exited (0) 7 months ago         unruffled_einstein

$ docker commit ecac70d6c2aa my-ubuntu
$ docker commit f08c0492d537 my-greeting
$ docker save -o /tmp/my-project.tar my-ubuntu my-greeting
$ tar -tf /tmp/my-project.tar  # lists the content of tar file

# transfer that file and load it in another system running docker service
$ docker load -i my-project.tar
$ docker images  # see if the project images are loaded
```

----
### 5. Uploading an Image to Dockerhub
1. First you should sign up for an account in docker hub (hub.docker.com).
2. Then tag your image compliant with your docker hub account.
3. Login, then push it!
```
$ docker tag f08c0492d537 omertb/helloworld:r1
$ docker login
$ docker push omertb/helloworld:r1
```

### 6. Some Troubleshooting
```
$ docker history <image-id-name>
$ docker top <container-id-name>
$ docker log <container-id-name>
$ docker inspect <container-id-name> | grep Links
$ docker inspect <container-id-name> | grep Pid

# finding out container ip address:
$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-id-name>
```
> In docker file, with RUN statement, add some cleaning commands to reduce image size:
```
RUN apt-get update && apt-get install -y openssh-server && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```
> To change docker0 bridge IP network:
```
$ systemctl stop docker
$ ip link del docker0
$ sed -i 's/\-\-bip\=172.16.30.0\/24/\-\-bip\=10.10.10.0\/24/g' /etc/default/docker

# The above command finds the expression containing "--bip=172.16.30.0/24" and changes it to --bip=10.10.10.0/24
# DOCKER_OPTS includes --bip expression.

$ systemctl start docker
$ ip a  | see if IP address of docker0 is changed.

```
> Setting hostname and sending stdout of a container to  syslog server
> _The Dockerfile for image of this container is available as "Dockerfile-Freeradius"._
```
$ docker run -d --hostname freerad_server --name freerad_ct --log-driver syslog \\
    --log-opt syslog-address=udp://172.16.100.100:514 -p 172.16.0.10:21812:1812/udp myservices:radius
```

