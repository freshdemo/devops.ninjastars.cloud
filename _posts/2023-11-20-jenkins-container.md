---
layout: post
title:  "Container - Jenkins"
date:   2023-11-20 21:27:07 -0400
tags: container infrastructure jenkins cicd
---

Jenkins is one of the most widely deployed CI tools. Here we’ll walk through getting the container setup.

## Step 1 - Install Docker

If you haven’t already start by installing Docker if you haven’t already, [![](Jenkins%20Container%20-%20Stephen%20Perciballi%20-%20Confluence/docs@2x.ico)Install Docker Desktop on Mac](https://docs.docker.com/desktop/install/mac-install/).

## Step 2 - Instantiate the Container

`docker run -p 8080:8080 -p 50000:50000 --restart=on-failure --volume $HOME/jenkins:/var/jenkins_home jenkins/jenkins`

A couple of notes on the above:

-   If you have another server running on port 8080 or simply want to change it, change the left side of the :, so 8888:8080. This will make your system run the service on port 8888, but still forwards to port 8080 inside the container so that you don’t have to change the networking configurations in Jenkins.
    
-   The `--volume $HOME/jenkins:/var/jenkins_home` will create a directory in your home directory called jenkins and map that inside the container. This is convenient for backups because you can pull a newer image of jenkins with docker pull jenkins/jenkins, re-run the docker run command, then destroy the original container and your system should be up to date with the configuration intact.
    
-   `jenkins/jenkins` pulls the official jenkins image from the jenkins user account on DockerHub.
    

## Step 3 - Get the Admin Credentials

The only account on the system is that of the user admin. There are a few ways to get it, listed from easiest to hardest:

1.  When you issue the docker run command in step 2 the logs should be displayed to your terminal, where you will see a block of text surrounded by 3 rows of \*\*\*'s with the password in the middle.
    
2.  From a terminal run `cat ~/jenkins/secrets/initialAdminPassword`. This will open the file in the mounted directory.
    
3.  Display the container logs by first figuring out the container ID by running `docker ps -a` and taking note of the 12 character ID at the start of each row. Then issue `docker logs <container ID>`.
    
4.  Finally you can go into the container to get it by first figuring out the container ID by running `docker ps -a` and taking note of the 12 character ID at the start of each row. Then issue `docker exec -it <container ID> /bin/bash`. Once inside the container issues `cat /var/jenkins_home/secrets/initialAdminPassword`.
    

## Step 4 - Initial Setup

For these steps I’ve found certain browsers will not be able to load the pages correctly, but it looks more like Jenkins is failing.

With the admin credentials handy,

1.  Navigate to [http://localhost:8080](http://localhost:8080/ "http://localhost:8080") (or change the port to whatever is on the left side of the : from step 2).
    
2.  Choose to install the recommended plugins.
    
3.  Create the user account. Note once this account is created the initial admin user account will no longer work.
    
4.  Follow any prompts to upgrade software and reboot Jenkins.
    

Be the first to add a reaction