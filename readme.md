# 95-702 Distributed Systems for Information Systems Management
# Lab 3 - Creating Containers and Deploying to the Cloud

This lab illustrates how to run a server in a container in the cloud. The application used is Interesting Picture from Lab 2. The two main tasks are to containerize that Java servlet application and then move it to the cloud. The functionality of the app will not change, only where it is running.

After this lab, you will be able to: create a local Docker container to run a servlet; create a Docker container in the cloud for that same servlet; understand basic Cloud terminology; and explain the trade-offs of using containers in the Cloud.

Complete Lab3_Quiz on Canvas as you work on this lab. See the checkered flags for references to the quiz questions.

___Docker___ is a technology for creating containerized applications - that is, your application runs inside a portable container that has all the elements needed to run that application. "Container" means a __process__ (running program) that executes on a machine. "Portable" means the container can be executed on your laptop or moved to another machine - like a cloud server. The container is isolated from other processes on the host machine, so if it crashes, it shouldn't take any other processes down. There are a few issues with isolation, though - for example, the ports that a regular process can access via sockets must be mapped from the container's internal (virtual) ports to the host's actual ports. Systems can be composed of multiple containers that typically use some other technology (like [Kubernetes](https://kubernetes.io/)) to talk to each other (instead of low level port access). See Docker's [documentation](https://docs.docker.com/) for more details. At the end of the lab, there's a reading assignment about Docker.

A Docker __image__ is an executable file created from a Dockerfile containing all the Docker commands needed to make the image plus a self-contained application. For example, a Web servlet must be packaged as a [war](https://en.wikipedia.org/wiki/WAR_(file_format)#:~:text=In%20software%20engineering%2C%20a%20WAR,that%20together%20constitute%20a%20web) file for use here. When the image is created, it will have an auto-generated name (like "admiring_tereshkova") - you can rename it if you'd like - and the image will have a separate UUID (unique identifier) and image name different than the auto-generated name.

The Docker commands used in the lab are all preceded by "docker"; they are case-sensitive; and when you create the Dockerfile, it ___must___ be in Linux-style format: no ".txt" on the end, and Unix line endings (see below for details).

This lab will get you to install Docker on your laptop, run Interesting Picture as a Docker image, then push that image to the cloud (we'll be using Codespaces). As you work through the commands, be sure to reflect on them by asking yourself these questions: What is each command's purpose? What software is being used by this command? How does this fit into a Distributed Systems context?

### Warning! If you are running Windows Home Edition, upgrade to Educational or Pro before doing this lab!

## Part 1: Running a Docker Image Locally

### 1.1 Install Docker

This part of the lab installs the Docker Desktop daemon on your laptop so that you can run Docker containers there.

1. Go to

https://docs.docker.com/install/  

and Install Docker CE (Community Edition) according to your system.  Scroll down to find links for Mac and Windows downloads.

2. After installation, make sure you have the Docker daemon running in the background. Open a **CMD window** (Windows) or **terminal** (Mac or Linux). Use the command

        docker ps

to check if it is running. (All commands are preceded by "docker".) You should see table headers with empty rows:

        CONTAINER ID    IMAGE   COMMAND CREATED STATUS  PORTS   NAMES

There are sometimes problems running Docker on Windows because it runs in a virtual machine.  ***Windows Note: if you don't see the above output and you're using Windows:***

>> From former TA Joseph Perrino:

>>If a Windows machine runs into an error where docker ps does not work after installing Docker and/or they get some kind of infinitely looping error that says there’s some issue with Docker Desktop when clicking on its settings, do the following:

>>Completely uninstall Docker (probably through Settings > Apps > Docker > Uninstall). This may also require removing references to Docker in the Windows Registrar.

>>Enable HyperV on Windows 10 using PowerShell via this link: https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v

>>Restart your computer. Run the commands in that link again to verify HyperV is installed. Follow the instructions in this link: https://andrewlock.net/installing-docker-desktop-for-windows/

***End of Windows note***

3. Run the Hello World test image to see if you Docker runs correctly:

        docker run hello-world

4. If you see the following message, that means you have installed Docker correctly.

        Hello from Docker!

        This message shows that your installation appears to be working correctly.

        To generate this message, Docker took the following steps:
        1. The Docker client contacted the Docker daemon.
        2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
        3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
        4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

        To try something more ambitious, you can run an Ubuntu container with:
         $ docker run -it ubuntu bash

        Share images, automate workflows, and more with a free Docker ID:
         https://hub.docker.com/

        For more examples and ideas, visit:
         https://docs.docker.com/get-started/

The command they suggest trying runs a container with an Ubuntu (linux) command shell; there's no need to run it, but try it if you'd like. If you do, you'll be running a linux shell - type "exit" to exit it.

5. Look at the Docker Desktop screen for the running Containers. In the CMD or terminal window, type

        docker container ls --all


### :checkered_flag: Answer question 1 on the Canvas quiz named Lab3_Quiz.

### 1.2 Creating a Custom Docker Container

1. Creating a war file (for all the details, see https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-ee-application.html)

**First**, in your file system (Windows FileManager or Mac Finder), create a directory (folder) called "docker"; ***and remember where it is***. Open a terminal (Mac) or CMD window (Windows) and cd to that directory.

**Second**, open the Lab 2 InterestingPicture project in IntelliJ. Follow these "if-else-if" directions:

**If you already see the file InterestingPicture-1.0-SNAPSHOT.war ...**

- ... somewhere in your project panel, you don't have to do the Build part below. The war file would be in the **target** directory (or possibly in the target/out directory) - you may have to expand target to see it. See Figure 1.

![Figure 1](figure1.png)

***Figure 1***

**else in IntelliJ, click Build->Build Artifacts. If one of the choices is "InterestingPicture:war" ...**

- ... click the arrow to get the sub-menu and click Build. As above, this should create the file InterestingPicture-1.0-SNAPSHOT.war, which will be in the target (or possibly target/out) directory.

**else** do the following:

- in IntelliJ, go to your InterestingPicture project and select File -> Project Structure.
	- click on Artifacts
	- click +, select Web Application:Archive, and select "For InterestingPicture: war exploded"
	- if the message "Library 'lib' required .. the artifact" and a Fix button appear, DO NOT CLICK on Fix.
	- click Create Manifest and agree to the suggested location
	- click OK in the Project Structure
	- choose Build -> Build Artifacts. Point to InterestingPicture:war and choose Build

**end if**

**Third**, copy the war file into your docker directory, but re-name it **ROOT.war**. You can do that with Finder (Mac) or File Explorer (Windows).

Alternatively, do it by hand: figure out what directory it's in:  
- right click on it to bring up its Properties, where you can see its directory path (or choose Reveal in Finder on a Mac).

Then do:
```
cp <path to the file>InterestingPicture-1.0-SNAPSHOT.war ROOT.war
```
in a terminal (Mac); if using a CMD window (Windows), use "copy" instead of "cp".

It is important that you name it ***exactly*** ROOT.war: it's case-sensitve.

### :checkered_flag: Answer question 2 on the Canvas quiz named Lab3_Quiz.

2. Creating a custom Docker container using Dockerfile

- in the docker directory, copy the file named **Dockerfile** (note: again, name it ***exactly*** like this, capitalized, and **no** extension) from github. ***DO NOT*** just download it - you'll get the HTML version of it, and that will not work; copy its contents. Save it to your docker directory. This file contains Docker commands to create a docker container with the temurin jdk21 and Tomcat. It then removes the default web app and copies your war file to the tomcat/webapps directory.

- save this file.

**If you’re using Windows**, do these two things:

-- ***make sure the file uses UNIX/Linux line endings*** if you're using Windows. In Notepad++, use Edit > EOL Conversion > UNIX Format.

-- ***make sure the file does NOT have a .txt extension*** if you're using Windows. For example, in Notepad++, choose Save As, type the name in quotes as "Dockerfile", change the File Type from Text Documents (\*.txt) to All Files (\*.\*), and click Save.

If you don't do this, Docker will not work. You'll also need to do this for the .sh file in Part 2.

### 1.3. Test your Docker image locally:

-In the **docker** folder, you should have the following structure:

    docker/
    ├── Dockerfile
    └── ROOT.war

- in a terminal window (Mac) or CMD window (Windows), in your docker directory, build the docker image with this command (note the space and period at the end); this will take a few seconds, and you'll see docker output scrolling by.

        docker build -t interesting_picture .

- you can see all of your images by running:

        docker images

It will display something like this (details will vary; "hello world" will likely be listed, too, but it's not shown here):

        REPOSITORY            TAG       IMAGE ID        CREATED          SIZE
        interesting_picture   latest    861ece9f7deb    2 minutes ago    108MB

- deploy the image in a container locally. The -p flag (for "publish") maps the actual port on your machine to the Docker container's port. If 8080 is in use on your laptop, change the first number to something else (say, 9000) and use that number in the browser instead.

        docker run --rm -it -p 8080:8080 interesting_picture

- open a browser window with the address:

        http://localhost:8080/

- you should see your app running. Test it to make sure it works correctly (again, use the port number from the run command if 8080 didn’t work for you). After making sure the app is running correctly, kill the program in the CMD or terminal window with ctrl-C.

---

### :checkered_flag: Answer question 3 on the Canvas quiz named Lab3_Quiz.

---


## Part 2: Running on the Cloud

Recall that the ***cloud*** is a fancy term for "someone else's servers". Some useful properties of cloud computing include not having to buy hardware, not needing to keep system software (like operating systems) up-to-date, not worrying about exactly where your application is running (although you should think about some possible downsides of that - for example, how is security handled in the cloud? What is the network latency of connecting to the application?), the ability to scale  - up or down - the number of servers in use based on current demand, and geographic placement to put servers closer to clients.

In this lab, you'll use [Codespaces](https://github.com/features/codespaces) which has a free tier for small-scale use. As you can see from the URL, it is part of github (owned by Microsoft) Other commercial cloud providers include [Amazon AWS](https://aws.amazon.com/), [Alibaba Cloud](https://us.alibabacloud.com/en), and [Codespaces](https://Codespaces.com)

### 2.1 Get started with the github
1. If you already have a github account, use that. If you do not have a github account, go to [github.com](https://github.com) and create one.

2. Create a new repository on github named Lab3; make it private. You can add a README file if you'd like, but it's not required.

3. In the Lab3 repo, click the Add file drop-down and choose Upload Files. Drag and drop your Dockerfile and ROOT.war from Part 1. At the bottom, click Commit Changes.

### 2.2 Get started with Codespaces
1. Go to [Codespaces](https://github.com/features/codespaces). You can browse this page and its links as needed. Click Get started for free.

2. Click New codespace. Click Select a repository, choose your Lab3 repo. Click main, US East, 2-core (the free mode), and Create codespace.

3. The files from Lab3 should show up on the left side. You should be able to view them. On the top of the left side, click the +file (it's an open square) icon (alternatively, click the three-line icon in the upper left; choose File->new text file), name the new file **devcontainer.json**. Edit the file to contain these lines:

```
{
    "build": {
        "dockerfile": "Dockerfile",
        "forwardPorts": [8080]
    }
}
```

Save the file; you many have to rename it after that.

4. <something something> When asked, Do you want to add Docker extensions?, answer yes. The Docker logo and some Information should appear.

5. Do the same Docker commands as in Part 1:

        docker build -t interesting_picture .
        docker images
        docker run --rm -it -p 8080:8080 interesting_picture

InterestingPicture should run in a browser window.        

6. Copy the address in the browser bar; it will have some wacky auto-generated name. Open a new browser tab and copy the address, just to see that this address works on its own.


## Part 3: Cloud and Containers Concepts

1. Read this article:

https://docs.docker.com/get-started/docker-overview/

2. Read "A Descriptive Literature Review and Classification of Cloud Computing Research", journal pages 36 through 39 (PDF pages 3 through 6).

http://aisel.aisnet.org/cgi/viewcontent.cgi?article=3672&context=cais

You must be on campus, or using the campus VPN, to view this article.

### :checkered_flag: Answer question 4 on the Canvas quiz named Lab3_Quiz.

### :checkered_flag: Answer question 5 on the Canvas quiz named Lab3_Quiz.
(If you don't know what version control is, see this: https://en.wikipedia.org/wiki/Version_control)

