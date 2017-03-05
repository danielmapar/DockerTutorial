# Docker Tutorial

## Introduction to Virtualization Technologies

![Screenshot](./images/pre%20virtualization.png)

![Screenshot](./images/hypervisor-based virtualization.png)


![Screenshot](./images/container-based virtualization.png)

  - Hypervisors approach handles many VMs with their own copy of the OS/Kernel and only manages resources at hardware level.

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

  - To remove an image just do: ```docker rmi ${imageName:tag}```
    - Ex: ```docker rmi danielmapar/flask-test```

## Dockerfile In Depth

  - Each ```RUN``` command will execute the command on the top writable layer of the container, then commit the container as a new image.

  - The new image is used for the next step in the Dockerfile. So each ```RUN``` instruction will create a new image layer.

  - It is recommended to chain the ```RUN``` instructions in the Dockerfile to reduce the number of image layers it creates.

  - When writing the ```RUN``` instructions, sort multi-line arguments alphanumerically

  - Instead of:

    - ```RUN a```
    - ```RUN a```
    - ```RUN c```

    ![Screenshot](./images/docker-file-1-image.png)


  - just do ```RUN a && b && c```

  ![Screenshot](./images/docker-file-2.png)

  - ```docker build -t danielmapar/debian-new-2:1.0 -f Dockerfile-2 .```

  ![Screenshot](./images/docker-file-2-image.png)

  - The ```CMD``` instruction specifies what command you want to run **when the container starts up**.

  - If we don't specify ```CMD``` instruction in the Dockerfile, Docker will use the default command defined in the base image

    - The default command for debian jessie is ```bash```


  - The ```CMD``` instruction doesn't run when **building** the image, it only runs when the container starts up

  - You can specify the command in either exec form which is preferred or in shell form

    - ```CMD["echo", "Hello World!"]```

  ![Screenshot](./images/docker-cmd-command.png)

  ![Screenshot](./images/docker-cmd-command-run.png)

  - You can also supply a command to be executed right after the container starts by running: ```docker run danielmapar/debian-new-3:1.0 ls /``` or ```docker run danielmapar/debian-new-3:1.0 echo "Test"```

  - You can also test it by doing: ```docker start -i ${containerName}```

## Dockerfile Cache

  - Each time Docker executes an instruction it builds a new image layer

  - The next time, if the instruction doesn't change, Docker will simply reuse the existing layer.

  ![Screenshot](./images/docker-cache.png)

  - ```docker build -t danielmapar/debian . --no-cache=true```
    - Using cache on ```apt-get update``` and ```apt-get install curl``` command can leave you with an older/cached version of ```curl``` instead of the latest

## COPY Instruction

  - The ```COPY``` instruction copies new files or directories from build context and adds them to the file system of the container.

  ![Screenshot](./images/docker-copy.png)

## ADD Instruction

  - The ```ADD``` instruction can not only copy files but also allow you to download a file from the internet and copy to the container.

  - ```ADD``` instruction also has the ability to automatically unpack compressed files

    - ```ADD http://example.com/big.tar.xz /usr/src/things/```
    - ```RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things```


  - The rule is: use ```COPY``` for the sake of transparency, unless you're absolutely sure you need ```ADD```.

  - For the ```ADD``` and ```COPY``` instructions, the contents of the file(s) in the image are examined and a checksum is calculated for each file. The last-modified and last-accessed times of the file(s) are not considered in these checksums. During the cache lookup, the checksum is compared against the checksum in the existing images. If anything has changed in the file(s), such as the contents and metadata, then the cache is invalidated.

## Push Images to Docker Hub

  - Create an account on https://hub.docker.com

  - Setup a repository.
    - Example: ```test_repo```

  ![Screenshot](./images/docker-hub-repo.png)

  - Tag one of your images with the proper repository:tag

    - ```docker images```

    - ```docker tag d5dc7cb2053d danielmapar/test_repo:1.00```

      - Tag will clone the image with the same image id, but with a new repository/tag.

  - Login to docker hub using your account

    - ```docker login```

  - Push the image

    - ```docker push danielmapar/test_repo:1.00```

- Note on Latest Tag

  - Docker will use latest as a default tag when no tag is provided

  - A lot of repositories use it to tag the most up-to-date stable image. However, this is still only a convention and is entirely not being enforced.

  - Images which are tagged latest will not be updated automatically when a newer version of the image is pushed to the repository.

  - Avoid using latest tag

## Dockerize a Hello World Web App

  - ```git clone -b v0.1 git@github.com:jleetutorial/dockerapp.git```
    - --branch/-b can also take tags and detaches the HEAD at that commit in the resulting repository.


  ![Screenshot](./images/docker-flask-dockerfile.png)

  - ```useradd -ms /bin/bash admin```
    - ```-m```: 	Create the home directory if it does not exist.
    - ```s```: User's login shell, which defaults to /bin/bash

  - ```USER admin```: This command ensures that we are running the app server under the ```admin``` user.
    - If you do not specify a user, it will be ran by the root user
    - An attacker could take advantage of being root

  - ```WORKDIR /app```: This instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions to follow it in the Dockerfile.

  - ```CMD ["python", "app.py"]```: Since we specified ```/app``` as an working dir, we can call app.py directly.

  - ```docker build danielmapar/flask-app:1.0 -f Dockerfile .```: Build image

  - ```docker run -d -p 5000:5000 danielmapar/flask-app:1.0```: Run container

  - ```docker exec -it ${containerId} bash```

  ![Screenshot](./images/inside-flask-container.png)

  - ```ps aux```:
    - a = show processes for all users
    - u = display the process's user/owner
    - x = also show processes not attached to a terminal

  ![Screenshot](./images/processes-inside-container-flask.png)


## Docker Container Links

![Screenshot](./images/docker-container-link.png)

### Implement Simple Key-Value Lookup Service

  - ```cd dockerapp```

  - ```git checkout v0.3```

  - ```docker build -t  danielmapar/python-flask-app:2.0 -f Dockerfile .```

  - ```docker run -d -p 5000:5000 danielmapar/python-flask-app:2.0```

  - ```docker logs ${containerId}```
    - You will see that the connection to the redis container failed. Lets create it

  - ```docker run --name redis redis:3.2.0```

  - ``docker run -d -p 5000:5000 --link redis danielmapar/python-flask-app:2.0```
    - The ```--link``` property will enable the app container to reach the redis container

  - ```docker logs ${containerId}```
    - Now the connection is stablished and the app is working


### How container links work behind the scenes?

  - ```docker exec -it ${containerId} bash```

  - ```more /etc/hosts```
    - When the text passed to it is too large to fit on one screen, it pages it. You can scroll down but not up.
    - ```less``` will let you scroll both ways

    - Ex: ```ps -ef | more```

    - You will see the redis container ip listed in the ```hosts``` file.
      - The ```hosts``` file will be hit before the DNS for lookups.

  - ```docker inspect ${redisContainerId} | grep IP```

  ![Screenshot](./images/docker-container-link-ip.png)

  - Let's ping redis inside the app container
    -  ```docker exec -it ${constainerId} bash```

    - ```ping redis```

  ![Screenshot](./images/docker-ping-redis.png)

### Benefits of Docker Container Link

  - The main use for docker container links is when we build an application with a microservice architecture, we are able to run many independent components in different containers.

  - Docker creates a source tunnel between the containers that doesn't need to expose any ports externally on the container.


## Docker Compose

  - Manual linking containers and configuring services become impractical when the number of containers grows

  - Their are 2 versions of the compose file format (both written using YAML)

    - Version 1, which is the legacy format does not support volumes or networks

    - Version 2, which is the most up-to-date format

  - docker-compose.yaml:

    ```
    version: '2'
    services:
        dockerapp:
          build: . // defines the path to the Dockerfile (this file is in the same directory)
          ports:
            - "5000:5000" // host port : container port
          volumes:
            - ./app:/app // A volume is a designated directory in a container (persistent even when the container is off). It can be used to share data between the host machine and the container (share source code is an example). This way, we don't need to use COPY on the Dockerfile for the source code. Format is -> host directory : container directory

        redis:
          image: redis:3.2.0 // We can either build an image, or run an existent image (this is obligatory)
    ```

    - You might want to ask, why we don't need to define the linking between app and redis. This is because the version 2 docker-compose format supports docker network feature, which allows containers to be discovered by its name automatically. Also, any service can reach any service at its service name. So linking is not required any more (if you are using version 2, version 1 requires it)

    - ```git checkout v0.4```

    - ```docker-compose up -d```: Will build all images and run all the containers
      - ```-d```: Run in detached mode

    - ```docker-compose start```: Will start the containers

    - Docker compose is a very handy tool to quickly get docker environment up and running

    - Docker compose uses yaml files to store the configuration of all the containers, which removes the burden to maintain our scripts for docker orchestration (such as: starting, linking, stopping)

## Deep Dive into Docker Compose Workflow

  - ```docker-compose ps```: To check the status and ids of containers managed by compose
    - ```docker exec -it ${containerId} bash```

  - ```docker-compose logs -f```: Will output both containers logs as the file is getting appended (```-f``` option).

  ![Screenshot](./images/docker-compose-logs.png)

  - ```docker-compose logs ${containerName}```
    - The container name is inside the docker-compose file

  - ```docker-compose rm```: To completely remove the containers

  - Even after removing the containers, docker keeps the image the way it is. Having said that, if I change the Dockerfile and run ```docker-compose up``` again, it will won't apply my changes.
    - To re-build the image, you need to run ```docker-compose build```

  ![Screenshot](./images/docker-compose-rebuild-1.png)

  ![Screenshot](./images/docker-compose-rebuild-2.png)

  ![Screenshot](./images/docker-compose-commands.png)

## Introduction to Docker Networking

  ![Screenshot](./images/docker-networking-default.png)

  - Docker uses the networking capabilities of the host machine OS to provide networking support for the containers running on the host machine.

  - Once the docker daemon is installed in the host machine, a network interface called docker0 is provisioned on the host, which will be used to bridge the traffic from the outside network to the internal containers hosted on the host machine.

  - Each container connects to the bridge network thought its container network interface. Containers can connect to each other and the outside world through this bridge network interface.

  - Docker Network Types:

    - Closed Network / None Network

    - Bridge Network

    - Host Network

    - Overlay Network

  - ```docker network ls```: Existing docker networks in host machine
    - Default is: ```bridge```, ```host``` and ```none```
