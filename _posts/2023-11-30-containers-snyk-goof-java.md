---
layout: post
title:  "Container - Snyk Goof Java"
date:   2023-11-30 21:27:07 -0400
categories: infrastructure container goof
---

This app can be built into containers that run on your laptop and GKE.

By building the container image you are able to demonstrate Snyk Container. The only reason to move beyond this step is if you need to demonstrate the same data from the Kubernetes integration.

## Step 1 - Build the Application

Follow this guide to install Java, get the code, and build the app, [Code Repository - Snyk Goof Java](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/17/1754431492).

## Option 1 - Laptop Container

This container will only run on Mac’s as the hardware architecture is different from most other systems you will encounter.

1.  From the local repo directory, `cd todolist-goof`. (From here you could run `docker-compose up` which will build the image and start a container. I tend to leave lots of artifacts around after using this so will continue this guide with `docker build`).
    
2.  `docker build -f ./Dockerfile -t <your DockerHub username>/<java-goof>:mac .`This will build a container locally and tag it with your registry username, a name of the image, and a tag for mac so you’ll know which image to use on your laptop versus running it on a different type of system. For example mine looks like `docker build -f ./Dockerfile -t freshdemo/java-goof-github-dockerhub:mac .` where I like to indicate the app name, where the code is stored, and where the container image is stored.
    
3.  List your images with `docker images` to verify you like the way it's tagged.
    
4.  Build a container to run the app with `docker run --rm -p 3002:8080 <your DockerHub username>/<java-goof>:mac`.
    
    1.  \--rm will run a container from your image, but destroy it when you cancel the command. If you remove --rm you will have a container that can be stopped/started whenever required.
        
    2.  \-p maps port 3002 on the outside (your laptop in this case) to port 8080 inside the container. You can set the external port to any other unused port on your system, but the 8080 needs to stay the same.
        
5.  You should now be able to access the app in a browser with `http://localhost:3002`
    
6.  Finally, manage the image and container artifacts left on your system. If there are containers based on the image you need to manage those first.
    
    1.  To push the image to the registry use `docker push <your DockerHub username>/<java-goof>:mac`. This is where the tag/name of the image is critical and will need to match the username on the registry. If you haven’t setup registry credentials this command should prompt you for them.
        
    2.  For a list of containers run `docker ps -a`. Take note of the container ID or name, and issue either a rm/start/stop, `docker container rm/start/stop <container ID>`
        
    3.  To remove the image run `docker images`. Take note of the image ID and issue a `container rmi <image ID>`
        

## Option 2 - Kubernetes Service Container

If you plan to run the container outside of your laptop the hardware architecture is likely different. This option is specific to building the container on your laptop, perhaps using Jenkins or Azure DevOps, and pushing to a registry to be consumed in a Kubernetes cluster.

1.  From the local repo directory, `cd todolist-goof`.
    
2.  Enable cross platform building on your Docker Desktop. These steps only have to be issued once per system, not container.
    
    1.  From `Docker Desktop` navigate to `Settings`, `Features in development`, `Experimental features`, and select `Access experimental features`.
        
    2.  Create a new builder instance with `docker buildx create --use`. This allows you to add the `buildx` and `--platform` flags seen below that are different from building for use locally.
        
3.  `docker buildx build --platform linux/amd64 --file ./Dockerfile --push -t <your DockerHub username>/<java-goof>:gke`. This will build a container locally, tag it with your registry username, a name of the image, and a tag for gke so you’ll know which image to use on your laptop versus running it on a different type of system. Also notice that it will `--push` the container in the same command. Because the multi-platform image won't run on your laptop it doesn't show up in the `docker images` list. The main difference you'll notice is specifying the platform as linux/amd64, where your laptop is arm64. For example mine looks like `docker buildx build --platform linux/amd64 -f ./Dockerfile -t freshdemo/java-goof-github-dockerhub:gke .`.
    
4.  Check your Docker Container Registry (i.e. [https://hub.docker.com](https://hub.docker.com/ "https://hub.docker.com")) to see that the image was pushed and is available.
    

## Option 3 - Build in a Pipeline

Jenkins and Azure DevOps don’t provide free accounts with cloud runners. This means all of the steps in the pipelines are run locally on the system you are using their runner software (likely your laptop), so you will use the docker build and push commands in the steps above while using these tools.

GitHub, Bitbucket, and Gitlab allow you to run build steps in their cloud and the steps vary. See the posts with the cicd tag to find specific use cases [https://snyksec.atlassian.net/wiki/label/~629db3cb76c0360069f263e7/cicd](https://snyksec.atlassian.net/wiki/label/~629db3cb76c0360069f263e7/cicd "https://snyksec.atlassian.net/wiki/label/~629db3cb76c0360069f263e7/cicd").

Be the first to add a reaction