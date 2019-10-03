# Docker Management

1. [Basic Commands](#1-basic-commands)
2. [Docker Networking](#2-docker-networking)
  - [Exposing Ports](#exposing-ports)
  - [Linking Directly](#linking-directly)
  - [Private Network](#private-network)


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
`$ docker rmi my-ubuntu` | Deletes the images named _my-ubuntu_.

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
$ docker run -d -P --name my-ssh-server eg_sshd
$ docker exec -ti my-ssh-server passwd root
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
If you check the content of /etc/hosts in the container ***server***, there will be seen an entry for client container.
> These types of links break when the containers exit.

#
#### Private Network

- This is the way to make links not break.
- This type of network has built in nameserver that fix the links.
- The private network must be created in advance

```
$ docker network create example-net
$ docker run --rm -ti --net=example-net --name server ubuntu bash
$ docker run --rm -ti --link server --net=example-net --name client ubuntu bash
```
