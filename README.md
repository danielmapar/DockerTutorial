# Docker Tutorial

## Introduction to Virtualization Technologies

![Screenshot](./images/pre%20virtualization.png)

![Screenshot](./images/hypervisor-based virtualization.png)


![Screenshot](./images/container-based virtualization.png)

  - Hypervisors approach is to handle many VMs with their own copy of the OS/Kernel and only manager resources at hardware level.

    - Porting your VM from one Hypervisor (Virtual box for example) to another one is not easy.


  - Container-based approach holds a single kernel and shares the OS with the containers.

      - Less CPU, memory and disk usage
      - Faster to boast
      - Portable


![Screenshot](./images/docker-architecture.png)

- Docker runs in a client / server architecture.

  - The daemon is the server (docker server, docker engine)
    - daemon is a computer program that runs in the background
  - The user interacts with the daemon by using the docker client
    - Client 1: Command line client
    - Client 2: Kitematic (Graphical client)

  - In a typical Linux installation, the docker client, daemon and containers run in the same host.

    - You can also connect a docker client to a remote docker daemon

  - Docker daemon can't run natively in Windows and Linux machines. That is because Docker daemon uses Linux specific kernel features. Having said that, we need to run Docker daemon inside a Linux VM to make it work.
    - Lightweight docker vm that was made to run the docker daemon (HYPERKIT)

      ![Screenshot](./images/mac-vs-linux.png)

    - Old installation process using docker-toolbox:

      ![Screenshot](./images/docker-toolbox.png)

      - Docker Client
      - Docker Machine
        - Lets you install docker engine in virtual hosts, and manage the hosts with docker machine commands.
      - Docker Compose
        - For defining and running multi container applications

      ![Screenshot](./images/docker-toolbox-installation.png)

      - ```docker-machine ls```: Lists all virtual machines running docker daemon

  - After installation you can use the docker client

    - ```docker info```: Outputs information about the docker environment (os, containers, images, etc)

    - ```docker version```: Prints the version of both docker client and docker server (docker daemon)


## Important Docker Concepts

  -  Images
    - Read only templates used to create containers
    - Are created with the ```docker build``` command, either by us or by other docker users
    - Are designed to be composed of layers of other images
    - Are stored in a Docker registry (Docker Hub)

  -  Container
    - If an image is a class, then a container is an instance of a class
    - They are a lightweight and portable encapsulations of an environment in which to run applications
    - We create a container from an Image. Inside that container we have all the binaries and dependencies we need to run our application.

  - Registry
    - Store our images
    - You can host your own registry, or use Dockers public registry (Docker Hub)
    - Inside a registry, images are stored in repositories
    - Docker repository is a collection of different docker images with the same name, that have different tags, each tag represents a version of the image (version of mysql for example)

  - Docker Hub (Registry)
    - New docker users are encouraged to use "official" repositories
      - Clear documentation
      - Docker incorporation dedicated a team for reviewing image content
      - Security updates in a timely manner

## Hello World Docker

  - When trying to create a container based on an image, Docker will first look for the image in the local box. If found, docker will use the local image. Otherwise, docker will download the image from the remote docker registry

    - ```docker images```: Show all local images available

    - ```docker run```: Creates the container based on the repository name and tag
    - ```docker run busybox:1.24 echo "Hello World"```: After spinning of the container running busybox version 1.24, docker will output "Hello World"

    - By running ```docker images``` again, you will now see budybox as part of your local images

    ![Screenshot](./images/docker-images.png)

    ![Screenshot](./images/docker-container-command.png)

  - To interact with the container and create a pseudo TTY (text-only console) we will use ```-i``` and ```-t``` respectively.

    - The ```-t``` flag creates a pseudo TTY that attaches ```stdin``` and ```stdout```


  - ```docker run -i -t busybox:1.24```: You will interact with the brand new container

  - **Important**: When you do ```docker -i -t run busybox:1.24``` as soon as you ```exit``` that container will no longer exist (including files, folder, etc..)
    - This is a foreground execution

    - If you run ```docker -i -t run busybox:1.24``` again, that will spin a completely new container with none of the previous files from the last execution.

    - Check running containers by doing ```docker ps```
      - ```ps```: Stands for processes snapshot

## Deep Dive into Docker Containers

![Screenshot](./images/docker-foreground-background.png)

  - You can use the ```-d``` option to run a container detached. That will output the container id.
    - ```docker run -d busybox:1.24```

  ![Screenshot](./images/docker-background.png)


  - ```docker ps -a```: The ```-a``` option stands for all containers. That includes the ones that got their execution stopped

  ![Screenshot](./images/docker-ps-a.png)

  - In case we want to remove the container after exiting it, we can do: ```docker run --rm busybox:1.24```
    - ```--rm```: Remove option

  - We can also specify the name of the container we want to run
    - ```docker run --name hello_world busybox:1.24```
    - If we do not specify a name, Docker will automatically generate one for you

    - ```docker inspect ${containerId}```: This command outputs some useful low level information, such as the image id and the log path.

- ```docker ps -a | awk '{ print $1,$2 }' | grep ${imageName:tag} | awk '{print $1 }' | xargs -I {} docker rm {}```: Run this command to remove all containers based on an specific image and tag -> ```${imageName:tag}```

  - AWK is a programming language designed for text processing and typically used as a data extraction and reporting tool. It is a standard feature of most Unix-like operating systems.

## Docker Port Mapping and Docker Logs

  - ```-p host_port : container_port```: Port option
    - ```docker run -it -d -p 8888:8080 tomcat:8.0```

  - ```docker logs ${containerId}```: That will enable you to see the container logs

  ![Screenshot](./images/docker-logs.png)


## Docker Image Layers

![Screenshot](./images/docker-image-layer.png)

  - An image is made off read only layers. The first layer is the base layer (the OS for example). The other layers will come on top of it (ex: Apache, emacs, etc..)

  - ```docker history busybox:1.24```: Busybox image consists of 2 layers, the first one adds a file and the second runs bash

  - Every container has a thin R/W layer on top of the layers that comes with the image. Having said that, when we ```run``` and image, it will spawn a new container with all those image layers + this unique thin writable layer attached to the container.
    - When we delete the container, the image and it's layers are kept the same way. However, the thin layer is deleted, since it was attached to the container.

    ![Screenshot](./images/container-thin-writable-layer.png)

      - This image shows a diversity of containers sharing the same image ```ubuntu:15.04```. In addition to that, it also shows that every single container has a Thin R/W layer on top of the underlying image layers.


## Build Docker Images

  - Approach 1 (Committing changes):

    - We can commit changes made in a Docker container.

    - Steps
      1. Spin up a container from a base image

        - ```docker run -it debian:jessie```

      2. Install Git package in the container

        - ```apt-get update && apt-get install -y git```
          - ```-y```: To confirm 'yes' to prompts

      3. Commit changes made in the container

        - ```docker commit ${containerId} ${repositoryName:tag}```

        - Docker commit command would save the changes we made to the Docker container file system to a new image.

        ![Screenshot](./images/docker-commit.png)
        ![Screenshot](./images/docker-new-repo.png)

        - ```docker run -it danielmapar/debian:jessie```: Inside the container you see git installed

  - Approach 2 (Writing Docker File):

    - A Dockerfile is a text document that contains all the instructions users provide to assemble an image

    - Each instruction will create a new image layer to the image

    - Instructions specify what to do when building the image

    - Instructions (not case sensitive):

      - ```FROM```: Specify the base image
      - ```RUN```: Any command you can run in a linux terminal

      ![Screenshot](./images/docker-file.png)

    - ```docker build -t ${repository:tag} ${path}```: Build an image using instructions given in a Dockerfile
      - ```docker build -t danielmapar/debian-new:1.0 .```
      - If we don't specify a tag, Docker will use 'latest' as a tag

    - Docker build also requires the path of the build context as an argument.

      - This is needed so Docker can copy source files from the local OS to the docker image

      - When build starts, docker client would pack all files in the build context into a tarball (.tar) then transfer the tarball file to the daemon.

      - By default, docker would search for the Dockerfile in the build context path

      - ```docker build [OPTIONS] PATH```: Path in this case will be ```.```, the current one
        - Docker will look for the Dockerfile inside the ```PATH```. However, you can specify the Dockefile location by also using the ```-f``` flag (```-file```).

      ![Screenshot](./images/docker-build-dockerfile.png)

      - Steps:

        1. The first step is to pack all the files from the the build context and send it to the daemon.

        2. Docker spins a container with debian:jessie.

        3. Docker runs a command inside that container.

        4. Docker commits the changes done in that container, this way generating a new image.

        5. Docker deletes the container.

        6. Docker spins a container based on that new image.

        7. Back to step 3. (command per command)

## Dockerfile In Depth

  - Each ```RUN``` command will execute the command on the top writable layer of the container, then commit the container as a new image.

  - The new image is used for the next step in the Dockerfile. So each ```RUN``` instruction will create a new image layer.

  - It is recommended to chain the ```RUN``` instructions in the Dockerfile to reduce the number of image layers it creates.

  - Instead of:

    - ```RUN a```
    - ```RUN a```
    - ```RUN c```

    ![Screenshot](./images/docker-file-1-image.png)


  - just do ```RUN a && b && c```

  ![Screenshot](./images/docker-file-2.png)

  - ```docker build -t danielmapar/debian-new-2:1.0 -f Dockerfile-2 .```

  ![Screenshot](./images/docker-file-2-image.png)
