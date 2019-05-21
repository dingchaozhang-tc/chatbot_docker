# Docker Demo for lunch and learn

# Docker Hello World
- Once you are done installing Docker, test your Docker installation by running the following:
```
docker run hello-world
```
To get started, let's run the following in our terminal:
```
docker pull ubuntu:12.04
```
The pull command fetches the ubuntu:12.04 image from the Docker registry and saves it to our system, we can see a list of all images using: 
```
docker images
```

The above gives a list of images that I've pulled from the registry, along with ones that I've created myself (we'll shortly see how). The TAG refers to a particular snapshot of the image and the IMAGE ID is the corresponding unique identifier for that image.

```
docker run ubuntu:12.04
```
Behind the scenes, a lot of stuff happened. When you call run, the Docker client finds the image (ubuntu:12.04 in this case), loads up the container and then runs a command in that container. When we run docker run ubuntu:12.04, we didn't provide a command, so the container booted up, ran an empty command and then exited. 
```
docker run ubuntu:12.04 echo "whats up"
```
All of that happened pretty quickly. Imagine booting up a virtual machine, running a command and then killing it. Now you know why they say containers are fast! 
The docker ps command shows you all containers that are currently running.
```
docker ps
docker ps -a
```
A way to run more than just one command in a container.
```
docker run -it ubuntu:12.04 sh
```
Running the run command with the -it flags attaches us to an interactive tty in the container. Now we can run as many commands in the container as we want. Leaving stray containers will eat up disk space. Hence, as a rule of thumb, clean up containers once we are done with them. To do that, you can run the docker rm command. Just copy the container IDs from above and paste them alongside the command.
```
docker rm $(docker ps -a -q -f status=exited)
```
This command deletes all containers that have a status of exited. the docker container prune command can be used to achieve the same effect.
```
docker container prune
```
# Terminology
- Images - The blueprints of our application which form the basis of containers. In the demo above, we used the docker pull command to download the ubuntu image.
- Containers - Created from Docker images and run the actual application. We create a container using docker run which we did using the busybox image that we downloaded. A list of running containers can be seen using the docker ps command.
- Docker Daemon - The background service running on the host that manages building, running and distributing Docker containers. The daemon is the process that runs in the operating system to which clients talk to.
- Docker Client - The command line tool that allows the user to interact with the daemon. More generally, there can be other forms of clients too - such as Kitematic which provide a GUI to the users.
- Docker Hub - A registry of Docker images. You can think of the registry as a directory of all available Docker images. If required, one can host their own Docker registries and can use them for pulling images.

# Our First Image
Tt's time to create our own. Our goal in this section will be to create an image that sandboxes a simple Flask application.  go ahead and clone the repository locally like so -
```
git clone git@github.com:dingchaozhang-tc/chatbot_docker.git
cd chatbot_docker/chatbot-master
```
The next step now is to create an image with this web app. We are going to use the python:3-onbuild version of the python image. These images include multiple ONBUILD triggers, which should be all you need to bootstrap most applications. The build will COPY a requirements.txt file, RUN pip install on said file, and then copy the current directory into /usr/src/app.
- Dockerfile
A Dockerfile is a simple text-file that contains a list of commands that the Docker client calls while creating an image. It's a simple way to automate the image creation process. The best part is that the commands you write in a Dockerfile are almost identical to their equivalent Linux commands.Go to chatbot-master and add a Dockerfile in the same directory where mian.py is in. We start with specifying our base image. Use the FROM keyword to do that -
```
FROM python:3-onbuild
```
The next step usually is to write the commands of copying the files and installing the dependencies. The onbuild version of the image takes care of that. We specify the port number that needs to be exposed. 
```
EXPOSE 5000
```
The last step is to write the command for running the application, which is simply - python ./main.py. We use the CMD command to do that -
```
CMD ["python", "./main.py"]
```
The whole Dockerfile should look like this:
```
# Instructions copied from - https://hub.docker.com/_/python/
FROM python:3-onbuild

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt


# tell the port number the container should expose
EXPOSE 5000

# run the command
CMD ["python", "./main.py"]

```
# Docker Build
Before you run the command yourself (don't forget the period), make sure to replace my username with yours. This username should be the same one you created when you registered on Docker hub. The docker build command is quite simple - it takes an optional tag name with -t and a location of the directory containing the Dockerfile.
```
docker build -t dingchao/chatbot .
```
The last step in this section is to run the image and see if it actually works (replacing my username with yours).
```
docker run -p 8888:5000 dingchao/chatbot
```
The command we just ran used port 5000 for the server inside the container, and exposed this externally on port 8888. Head over to the URL with port 8888, where your app should be live.

# Docker push
The first thing that we need to do before we deploy our app to AWS is to publish our image on a registry which can be accessed by AWS. There are many different Docker registries you can use (you can even host your own). For now, let's use Docker Hub to publish the image. To publish, just type

```
docker push dingchao/chatbot
```

# EC2 Deployment 
Now that your image is online, anyone who has docker installed can play with your app by typing just a single command.
- Go to EC2 in AWS, configuring inboud ports in security group:
Go to the "Network & Security" -> Security Group settings in the left hand navigation,find the Security Group that your instance is apart of Click on Inbound Rules,u se the drop down and add HTTP (port 80). 
and also choose the right VPC group used in this demo in developer_qa environment.
First we install docker on EC2:
```
sudo apt install docker.io
```
If you’re using a different Linux distribution other than Ubuntu, you may need different installation command line.
Once docker is installed, run the following command to start the app.
```
docker run -d -p 80:5000 dingchao/chatbot
```
Copy paste the public ip address of the EC2 instance into the browser, the app should be alive there.
- Stop and prune the containers
```
docker stop $(docker ps -a -q)
```
# Next MULTI-CONTAINER ENVIRONMENTS

Why multi-container?one of the key points of Docker is the way it provides isolation. The idea of bundling a process with its dependencies in a sandbox (called containers) is what makes this so powerful. Just like it's a good strategy to decouple your application tiers, it is wise to keep containers for each of the services separate. Each tier is likely to have different resource needs and those needs might grow at different rates. By separating the tiers into different containers, we can compose each tier using the most appropriate instance type based on different resource needs. 
