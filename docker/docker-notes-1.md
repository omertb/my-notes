# Docker Management Command

### 1. Basic Commands

Command | Description
------- | -----------
`$ docker images` | Lists images installed on local system.
`$ docker ps` | Lists running containers.
`$ docker run -ti ubuntu bash` | Starts a container running bash from ubuntu image. An interactive bash terminal is connected in foreground.
`$ docker run -d -ti ubuntu bash` | -d: starts the container in background. To bring the container foreground: `docker attach CONTAINER-NAME`
`docker ps -a` | Lists all container including exited ones.
`docker ps -l` | Shows the last exited container.
`docker commit sleepy-einstein my-ubuntu` | Converts the container named "sleepy-einstein" to a image named "my-ubuntu".
`docker exec -ti sleepy-einstein bash` | Runs bash (or any other) process interactively in the container sleepy-einstein.
`docker run --name example -d ubuntu bash -c "cat /etc/passwd"` |  Runs the bash command (cat /etc/passwd) in the container named "example".
`docker logs example` | Prints the output of the command in the created container above, that is, content of _/etc/passswd_.
`docker rm example` | Deletes the container named _example_.
`docker rmi my-ubuntu` | Deletes the images named _my-ubuntu_.

> To detach container console without exiting, press **ctrl+p, then q**.

