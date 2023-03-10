# Docker Commands
[https://docs.docker.com/engine/reference/commandline/cli/](https://docs.docker.com/engine/reference/commandline/cli/)
# Container Management
command | Description
---|---
`docker ps`| shows only running containers
`docker ps -a`| shows all containers (both stopped and running)stopped containers are listed with a status of 'exited'
`docker create <image>`| create a container without starting it
`docker rename <container-name> <new-name>`| renames a container
`docker container prune`| removes ***ALL*** stopped containers
`docker rm <containerName1> <containerName2>`| removes specific stopped containers
`docker port <container-name>`| show the port mapping of a container
`docker stats <container-name>`| show live resource usages stats for a container

# Running Containers
command | Description
---|---
`docker run <image>` | Example: `docker run hello-world`creates a container using the [hello-world image](https://hub.docker.com/_/hello-world) from DockerHub
`docker run -it <image> <command>`| Example: `docker run -it ubuntu /bin/bash`creates a container using the [official ubuntu image](https://hub.docker.com/_/ubuntu) from DockerHub. If the latest image is not found locally, It may initially return a message saying: `unable to find image Ubuntu:latest locally` and then automatically pull the ubuntu image from the DockerHub registry. The `-it` flags provide an interactive terminal session on the container. `i` makes it interactive and `t` implements a pseudo TTY (i.e makes the container terminal behave like a standard terminal).
`docker run --name <container-name> <image>`| create start and name a container
`docker run -p <host>:<container-port> <image>`| maps a host port to a container port
`docker start <container-Name>`| start a stopped container (get the name from `docker ps -a`)By default the container is run in the background, use `docker attach` to interact with it
`docker attach <container-name>`| allows you to interact with a running container
`exit`| exit and stop an attached container
`docker stop <container-name>`| stop a running container
`docker run -d <image>`| runs a container in detached mode (so in the background)
# Networking
command | Description
---|---
`docker network ls`| view available networks
`docker network rm <network>`| remove a network
`docker network inspect <network>`| show info about a network (subnets, IPV6enabled etc.) using either network name or id
`docker network connect <network> <container>`| connect a container to a network
`docker network disconnect <network> <container>`| disconnect a container from a network
# Image Management
Command | Description
---|---
`docker images`| lists out the docker images installed locallydisplays the image name, the tag, , and the image ID
`docker rmi <image>`| removes a specific image using the Image ID or name
`docker image inspect --format='' <image>`| View an image's labels (key-value pairs)
`docker image prune`| removes ***ALL*** unused images
`docker build -t <name>:<tag> .`| Create an image from a Dockerfile (see section below)note:`.` assumes you are in the directory containing the Dockerfile, you can replace this with a relative pathname
# Creating your own Image using a Dockerfile
A Dockerfile (captialized D, one word, no file extension) is a text file containing commands you can use to create your own image

## Basic syntax for a Dockerfile
This is not a exhaustive list see: [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/) for a full overview

Example contents of a Dockerfile could look like

```
# using the empty image named 'scratch' - https://hub.docker.com/_/scratch
FROM scratch
# copies the folder hello
COPY hello /
# executes the executable file when a container is created using the image we created from the Dockerfile
CMD ["/hello"]
```

## `FROM`
The starting image that you want to build on top of`FROM` must be the first instruction in the Dockerfile
Example Usage

  * `FROM <image>`
  * `FROM <image>:<tag>`
  * `FROM <image>@<digest>`

## `COPY`
Files or directories to copy from your source directory into the root directory of the destination image (wildcards are supported)
## `RUN`
Runs a command and commits the result
Example usage:

  * Shell form: `RUN <command>`Example: `RUN /bin/bash -c 'source $HOME/.bashrc && echo $HOME'`
  * Exec Form: `RUN ["<executable>", "<param1>", "<param2>"]`Example: `RUN ["/bin/bash", "-c", "echo hello"]`

## `EXPOSE`
Indicates what port the container will be listening on, it doesn't automatically open the port on the container

## `CMD`
The default command to run when a container is made from the image, usually launching the software required in a container. For example, the user may need to run an executable `.exe` file or a Bash terminal as soon as the container starts
There should be only one command instruction per Dockerfile (if you include multiple only the bottom most command will be used)

Example Usage:

```
CMD ["executable", "parameter1", "parameter2"]
```
## `LABEL`
Adds labels to an image in the form of key-value pairse.g. `LABEL version="1.0" other="value3"`

# Build the image from Dockerfile
To build the image using the docker file run:

```
docker build -t <name-for-your-image> .
```
you can then create a container using:

```
docker run <image-name>
```
# Create Images from Containers
You can create images from containers (although Dockerfiles are the preferred method as they are easily shareable and can be committed to source control).

```
docker commit <container-id> <new-image-name>
```

create an image from a container with a small change

```
docker commit --change'CMD ["python", "-c", "import this"]' <contiainer-id> <new-image-name>
```
# Port Mapping
You can retrieve the container's IP using:

```
docker inspect <container-id-or-name>
```
by default they are all running on port 8080 so you can interact with each other via the container's IP address.

use `EXPOSE` in the Dockerfile to define a port to communicate with your application running in your container (e.g. `EXPOSE 8080`) but note this doesn't do anything to the host.

To interact with a web application inside of a container, you can bind the port in the container to a port on the host (which is running the containers).
The mapping can be done dynamically with the publish all flag (`-P`) or explicitly with the publish flag (`-p` ).

The `-P` (uppercase) flag is the short form of `publish all` and this will dynamically mapp the exposed ports of the container to open ports on the host.

For example:

```
docker run -d -P <image>
```
will run the container in detached mode (in the background), and also dynamically bind the exposed port 8080 to a port on the host.
You can then use the `docker ps -a` command to see which port.
In this example: `0.0.0.0:32768 ->8080/tcp` the port which was dynamically exposed is 32768.

this means you can curl `localhost:32768` and interact with the application

You can also use the publish flag, which will allow you to map specific ports using the `-p` (lowercase) flag, which is the short form of publish.
so `docker run -d -p 3000:8080` maps port 3000 on the host to port 8080 inside the container.
