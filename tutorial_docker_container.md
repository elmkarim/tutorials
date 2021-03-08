# A step-by-step guide to creating Docker containers
 
## 1. What is a Docker container
A Docker container is a lightweight package that allows to include all files required to run applications. A Docker container has many advantages compared to a virtual machine (VM). While a VM requires its own OS and CPU/RAM, a Docker container uses the host OS and hardware. It also allows us to store all files needed by an application in one container (see figure below).
 
![container vs VM](./img/Container_VM_Implementation.png "Title")

A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. The container includes everything needed to run an application: code, runtime, system tools, system libraries, and settings.
 
Refer to [this page](https://www.docker.com/resources/what-container) for more details on containers. 
 
## 2. Docker engine installation
Refer to [this page](https://docs.docker.com/engine/install/) for instructions on Docker installation on your machine. In what follows, we suppose we are using a Linux machine.
 
## 3. Installing a machine learning image
We can create our own image or download a ready-to-use image from [Docker hub](https://hub.docker.com/). Let's say we are interested in a container for machine learning with a jupyter notebook web interface and we found a good image: `jupyter/datascience-notebook` on Dockerhub. The first step is to pull this image to our local machine then run a container based on it. 
 
Pull the image with:
```sh
docker pull jupyter/datascience-notebook 
```
 
Once done, check for the existing images:
```sh
docker images
```
Each image (or repository) has an unique id used to refer to in any operation. The image name can also be used in association with its tag if different versions exist locally. In the example below, we also see a mysql image pulled previously.
```
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
jupyter/datascience-notebook   latest              53a378c64e80        42 hours ago        3.96GB
mysql                          5.7                 ae0658fdbad5        3 months ago        449MB
```
 
## 4. Running a container from an image
The following command shows a simple way of running a container from an image. Note that the tag `:latest` is optional here.
```sh
docker run jupyter/datascience-notebook:latest
```
This method is not efficient as the container will only run as long as our terminal is active. We can read the container's output on the terminal, in our case it is jupyter notebook server log. So we will go ahead and stop the execution by `CTRL-C`, then show the history of containers:
```sh
docker ps -a
```
We can see that the jupyter container has exited, but is still stored locally for further usage. The mysql container is up and running and does not interfere with the jupyter container.
```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                     PORTS                                            NAMES
170fa67cf91f        jupyter/datascience-notebook:latest   "tini -g -- start-no…"   7 minutes ago       Exited (0) 6 minutes ago                                                    eloquent_ritchie
0740e5d761a5        mysql:5.7                             "docker-entrypoint.s…"   2 months ago        Up 2 months                0.0.0.0:3306->3306/tcp, 33060/tcp                sb-mysql
```
To restart the jupyter container, we can use its `CONTAINER ID` listed above or the "weird" name given by Docker `eloquent_ritchie`.
```sh
docker start eloquent_ritchie
```
Check again the running containers:
```sh
docker ps -a
```
```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                    PORTS                                            NAMES
170fa67cf91f        jupyter/datascience-notebook:latest   "tini -g -- start-no…"   20 minutes ago      Up 33 seconds             8888/tcp                                         eloquent_ritchie
0740e5d761a5        mysql:5.7                             "docker-entrypoint.s…"   2 months ago        Up 2 months               0.0.0.0:3306->3306/tcp, 33060/tcp                sb-mysql
```
We will see in the next section how we can start a bash session on a container to run linux commands. But for the moment, let's see how to use the existing container. Let me remind you here that a container uses the host resources and OS, so if we run `ps -ef` locally, we will list all processes owned by containers. We shall communicate with a container through the ports it exposes (found on the column `PORTS` above). The jupyter container exposes port 8888, however as we have not mapped this port to any local port, it is not accessible. There are various ways to reach this port, the easiest method is to map the container port 8888 to a local port.
 
So let's stop and remove the existing container then create a new one with the proper port mapping.
```sh
docker stop eloquent_ritchie
docker rm eloquent_ritchie
```
The command below creates a new container named `jupyter` and maps the container port 8888 to the local port 5000, we can very well use the same port 8888 locally if available:
```sh
docker run -d -p 5000:8888 --name jupyter jupyter/datascience-notebook:latest
```
The `-d` option runs the container in detached mode (in the background), `-p hostPort:containerPort` maps the host and container ports, and `--name` gives a name to our container. Let's check the running containers:
```sh
docker ps -a
```
```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                    PORTS                                            NAMES
ea9d6eb28602        jupyter/datascience-notebook          "tini -g -- start-no…"   15 seconds ago      Up 14 seconds             0.0.0.0:5000->8888/tcp                           jupyter
0740e5d761a5        mysql:5.7                             "docker-entrypoint.s…"   2 months ago        Up 2 months               0.0.0.0:3306->3306/tcp, 33060/tcp                sb-mysql
```
 
## 5. Accessing a container
### 5.1. Through the exposed ports
Each container exposes one or many ports that can be mapped to arbitrary host ports. The jupyter container that we created in the section above exposes the port 8888 that we mapped to the host port 5000. 
Let's check if the port is accessible with the Linux `curl` command. 
```sh
curl localhost:5000
```
We don't get any error message, which suggests the jupyter container is working. If the local Linux machine has a web browser, we can type the url `localhost:5000` to display the jupyter notebook's web interface, or access it from another machine if port 5000 is open in the firewall by typing `Machine_IP:5000` on the web browser. When launched for the first time, jupyter asks for a token code that is found on the jupyter web server output. Although we ran the jupyter container in detached mode, we can easily read its output with this command:
```sh
docker logs jupyter
```
```
Executing the command: jupyter notebook
...
[I 16:17:44.810 NotebookApp] Jupyter Notebook 6.1.4 is running at:
[I 16:17:44.810 NotebookApp] http://ea9d6eb28602:8888/?token=2ebc6400e5145eab6ce9cf006eb12830b1db2529024f6d1f
[I 16:17:44.810 NotebookApp]  or http://127.0.0.1:8888/?token=2ebc6400e5145eab6ce9cf006eb12830b1db2529024f6d1f
[I 16:17:44.810 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 16:17:44.816 NotebookApp]
 
    To access the notebook, open this file in a browser:
        file:///home/jovyan/.local/share/jupyter/runtime/nbserver-6-open.html
    Or copy and paste one of these URLs:
        http://ea9d6eb28602:8888/?token=2ebc6400e5145eab6ce9cf006eb12830b1db2529024f6d1f
     or http://127.0.0.1:8888/?token=2ebc6400e5145eab6ce9cf006eb12830b1db2529024f6d1f
...
```
And here we find the token!
 
### 5.2. Through commands
We often need to run a particular command on the container, the docker exec command allows us to do this. Let's run `pwd` to get the current directory.
```sh
docker exec jupyter pwd
```
```
/home/jovyan
```
This folder only exists in the container. We can use the same method to run a shell instance, only this time, we will add the modifier `-it` to run it in interactive mode, otherwise, the shell will run and exit immediately. Below we type some Linux commands inside the container:
```sh
docker exec -it jupyter bash
```
```
(base) jovyan@ea9d6eb28602:~$ ls
work
(base) jovyan@ea9d6eb28602:~$ cd work
(base) jovyan@ea9d6eb28602:~/work$ mkdir project1
(base) jovyan@ea9d6eb28602:~/work$ ls
project1
(base) jovyan@ea9d6eb28602:~/work$ whoami
jovyan
(base) jovyan@ea9d6eb28602:~/work$exit
```
It is interesting to see that the user `jovyan` is only defined inside the container. Sometimes this default user may not have root privileges which results in the rejection of all sudo operations. If sudo is needed, we have to run the instance with the root user and grant full sudo privileges. 
```sh
docker run -d -p 5000:8888 --user root -e GRANT_SUDO=yes --name jupyter jupyter/datascience-notebook:latest
```
With machine learning containers, it is useful to run shell commands occasionally to install a new package as in the example below.
 
```sh
docker exec jupyter pip install nltk
```
```
Collecting nltk
  Downloading nltk-3.5.zip (1.4 MB)
Requirement already satisfied: click in /opt/conda/lib/python3.8/site-packages (from nltk) (7.1.2)
Requirement already satisfied: joblib in /opt/conda/lib/python3.8/site-packages (from nltk) (0.17.0)
Collecting regex
  Downloading regex-2020.11.13-cp38-cp38-manylinux2014_x86_64.whl (738 kB)
Requirement already satisfied: tqdm in /opt/conda/lib/python3.8/site-packages (from nltk) (4.50.0)
Building wheels for collected packages: nltk
  Building wheel for nltk (setup.py): started
  Building wheel for nltk (setup.py): finished with status 'done'
  Created wheel for nltk: filename=nltk-3.5-py3-none-any.whl size=1434674 sha256=566797c15829048734e842e22a0d713fb158834e7498109a5a2fbb173187cad2
  Stored in directory: /home/jovyan/.cache/pip/wheels/ff/d5/7b/f1fb4e1e1603b2f01c2424dd60fbcc50c12ef918bafc44b155
Successfully built nltk
Installing collected packages: regex, nltk
Successfully installed nltk-3.5 regex-2020.11.13
```
Another way would be to run an interactive bash instance with `docker exec -it jupyter bash`, then execute `pip install nltk` on the command line.
 
## 6. Using volumes for persistent storage 
All files created inside a container will only exist inside it, thus, we should be very careful before removing any container as this will also remove the files. Even though the jupyter web interface allows uploading and downloading files, we certainly would not like to lose our project files if we mistakenly remove the container. A convenient way for keeping our files secured is to store them in an attached volume. There are many ways to bind volumes to a container explained [here](https://docs.docker.com/storage/volumes/). A volume can be created in Docker and used to safely share data between containers. Data in a volume is persistent and is independently managed by Docker. It can be backed up and migrated easily. 
The simplest way to bind a volume to a container is at container creation.
Now, let's create a volume as our working directory in the jupyter container. 
```sh
docker volume create jupyter_volume
```
Check if the volume is created:
```sh
docker volume ls
```
```
DRIVER              VOLUME NAME
...
local               jupyter_volume
...
```
Inspecting a volume reveals its local path (`/var/lib/docker/volumes/jupyter_volume/_data`). This is particularly useful for backup or offline editing.
```sh
docker volume inspect jupyter_volume
```
```
[
    {
        "CreatedAt": "2021-03-06T18:45:53Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/jupyter_volume/_data",
        "Name": "jupyter_volume",
        "Options": {},
        "Scope": "local"
    }
]
```
 
Let's stop and remove the current jupyter container:
```sh
docker stop jupyter
docker rm jupyter
```
Then create a new one to which we bind the newly created volume `jupyter_volume` to the container's folder `/home/jovyan/work/notebooks`:
```sh
docker run -d -p 5000:8888 --name jupyter -v jupyter_volume:/home/jovyan/work/notebooks jupyter/datascience-notebook:latest
```
We will now create a test file inside the local volume folder (the folder may be different in your machine, use the `volume inspect` docker command to show the local volume location):
```sh
cd /var/lib/docker/volumes/jupyter_volume/_data
touch test.txt
```
We can quickly check the presence of the file on the container:
```sh
docker exec jupyter ls -l /home/jovyan/work/notebooks/
```
```
total 0
-rw-r--r--. 1 root root 0 Mar  6 19:45 test.txt
```
Note that trying to write files from the container to the volume may result in a write permission error. A discussion of this issue can be found [here](/var/lib/docker/volumes/jupyter_volume/_data). 
```sh
docker exec jupyter touch /home/jovyan/work/notebooks/test2.txt
```
```
touch: cannot touch '/home/jovyan/work/notebooks/test2.txt': Permission denied
```
Many solutions are suggested, one of them is to change the write permission on the host volume:
```sh
chmod 777 /var/lib/docker/volumes/jupyter_volume/_data
docker exec jupyter ls /home/jovyan/work/notebooks
```

```
test2.txt
test.txt
```
Finally, to provide persistent storage, we can simply bind a host folder to a container folder using the same `-v` option. The following command binds the local folder `/root/jupyterhub` to the container folder `home/jovyan/work/notebooks`.
```sh
docker run -d -p 5000:8888 --name jupyter -v /root/jupyterhub:/home/jovyan/work/notebooks jupyter/datascience-notebook:latest
```
 
## 7. How to create a customized image
Images that we download from Docker hub often serve as base to our project. After adding files and packages, our image becomes ready for deployment. There are two ways to create an image. The first one is to convert an existing container into an image, alternatively we write a docker file that starts with a base image and specifies the required transformations to produce the new image.
### 7.1. Converting a container into an image
Let's consider the `jupyter` container created above. We will install the `nltk` package and copy the content of a local folder to prepare a new image:
```sh
docker exec jupyter pip install nltk
docker exec jupyter mkdir /home/jovyan/templates
docker cp ~/nltk_data/. jupyter:/home/jovyan/templates  #copy all files from local folder nltk_data to container folder /home/jovyan/templates
```
Now let's generate a new image and name it `jupyter_nltk`.
```sh
docker commit jupyter jupyter_nltk
docker images
```
```
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
jupyter_nltk                   latest              dedb4af35544        2 seconds ago       3.99GB
...
```
At this step, we will stop the original container `jupyter` and remove it. 
```sh
docker stop jupyter
docker rm jupyter
```
It is easy now to run a container based on our fresh `jupyter_nltk` image. This new container will have the `nltk` package and the `templates` folder present by default.
```sh
docker run -d -p 5000:8888 --name jupyter jupyter_nltk
```
With a docker hub account, you can follow the instructions [here](https://docs.docker.com/docker-hub/) to upload the newly created image and pull it from any machine.
 
### 7.2. Creating an image from a dockerfile
A dockerfile encloses all the necessary steps to create a modified image from a base image. The dockerfile below is the one I used to create a nodejs server image for the forge dashboard app. We start by making a folder `smartbuildingapp` then we add a file named `dockerfile` with any text editor.
```sh
mkdir smartbuildingapp
cd smartbuildingapp
vi dockerfile
```
Paste the following content:
```sh
# Base image to pull
FROM node 
# Create a new directory on the image and make it the work directory
RUN mkdir -p /home/elmkarim/dashboard 
WORKDIR /home/elmkarim/dashboard 
# Expose port 3000 (default nodejs port)
EXPOSE 3000  
# Run node with start.js as parameter 
CMD ["node", "start.js"]
```
Note that this image does not contain any script files for nodejs to work, the web content will be attached as a volume when running the container.
Now it is time to create our image from `dockerfile`:
```sh
docker build  . -t smartbuildingapp 
```
Check if the image was created successfully:
```sh
docker images
```
```
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
smartbuildingapp               latest              05ca2bf12f4c        8 seconds ago       937MB
...
```
To commit this image to dockerhub for easy deployment, please follow the instructions [here](https://docs.docker.com/docker-hub/).
 
## 8. Running multiple containers with docker compose
Compose is a tool for defining and running multi-container docker applications. We create a Linux YAML file to configure the application's services. We create and start the services from this configuration file.
The starting point is to install Compose by following the instructions [here](https://docs.docker.com/compose/install/).
In the host machine, we create a folder named `dashboard` and copy all the web content required by nodejs into it. Later, this folder will be mounted as a volume on the nodejs container.
 
Add a file named `docker-compose.yml` and paste the content below into it. This file instantiates three containers for the forge dashboard. 
```sh
version: "3.8"
 
# Listing our three containers. Or "services", as known by Docker Compose.
services:
    # Defining our MySQL container.
    # "mysql" will be the network alias for this container.
    mysql:
        image: mysql:5.7
        container_name: sb-mysql
        networks:
            - sb-network
        ports:
            - "3306:3306"
        volumes:
            - mysql_volume:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: <root_password_here>
            MYSQL_USER: BAS_DEV_RW
            MYSQL_PASSWORD: <mysql_password_here>
            MYSQL_DATABASE: bas_dev
 
    # Defining our Elasticsearch container
    # "elasticsearch" will be the network alias for this container.
    elasticsearch:
        image: elasticsearch:7.8.1
        container_name: sb-elasticsearch
        networks:
            - sb-network
        ports:
            - "9200:9200"
            - "9300:9300"
        volumes:
            - esdata1:/usr/share/elasticsearch/data:rw
        environment:
            discovery.type: single-node
            ES_JAVA_OPTS: -Xms256m -Xmx512m
 
    app:
        image: elmkarim/smartbuildingapp
        container_name: sb-app
        networks:
            - sb-network
        ports:
            - "3000:3000"
        volumes:
            - ./:/home/elmkarim/dashboard/
 
# The volume that is used by the MySQL container
volumes:
    mysql_volume:
    esdata1:
 
# The network where all the containers will live
networks:
    sb-network:
```
The three instantiated containers (or services) are:
 - `mysql`: the mySQL server that contains the relational building ontology model,
 - `elasticsearch`: the Elasticsearch server for time series data,
 - `app`: the nodejs web server that displays the forge dashboard and sends queries to mysql and elasticsearch from user interaction.
 
The three containers are connected through an internal network `sb-network` and use two volumes: `mysql_volume` and `esdata1` for persistent storage. It is worth mentioning that the `sb-network` provides a unique IP address for each container that is different from the host IP.
 
The host folder where `docker-compose.yml` exists is bound to the working directory of the `app` container. This allows more flexibility in terms of editing web files and creating backups.
 
To run the three containers in the background, use this command:
```sh
docker-compose up -d
```
The main application is the nodejs web server that is accessible through `localhost:3000`. Elasticsearch and mysql can also be reached through their respective ports on localhost.
 
We can selectively stop any container using `docker stop <container_name>`, or stop them all at once by:
```sh
docker-compose down
```

## References
- [What is a Docker container](https://www.docker.com/resources/what-container)
- [Docker engine installation](https://docs.docker.com/engine/install/)
- [Docker hub](https://hub.docker.com/)
- [Docker volumes](https://docs.docker.com/storage/volumes/)
- [How to commit and image to docker hub](https://docs.docker.com/docker-hub/)
- [Converting a container into an image](https://www.scalyr.com/blog/create-docker-image/)
- [What is Docker-compose](https://docs.docker.com/compose/)
- [How to install Docker compose](https://docs.docker.com/compose/install/)