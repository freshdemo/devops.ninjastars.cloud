---
layout: post
title:  "Build Containers & Push to Registry"
date:   2023-11-16 21:27:07 -0400
tags: infrastructure containers registry
---
Often the code you’re using the demo will have a pre-build container image hosted on [https://hub.docker.com](https://hub.docker.com/ "https://hub.docker.com"), for example [https://hub.docker.com/r/bkimminich/juice-shop](https://hub.docker.com/r/bkimminich/juice-shop). This allows you to pull a theoretically working container to your laptop, a VM, kubernetes, or anywhere else you want to run it.

You may want to build your own container to include vulnerabilities, remove vulnerabilities, change pages, add/remove users, add/remove tags that can be used with Insights, and may more.

## Step 1 - Install Docker

Start by installing Docker if you haven’t already, [![](Build%20Containers%20&%20Push%20to%20Registry%20-%20Stephen%20Perciballi%20-%20Confluence/docs@2x.ico)Install Docker Desktop on Mac](https://docs.docker.com/desktop/install/mac-install/).

## Step 2 - Create a Registry Account

DockerHub is currently the best option because they offer SaaS Container Registry services free. Create an account on [![](Build%20Containers%20&%20Push%20to%20Registry%20-%20Stephen%20Perciballi%20-%20Confluence/favicon.ico)Docker Hub Container Image Library | App Containerization](https://hub.docker.com/).

## Step 3 - Build a Container

Change into the juice-shop directory on your laptop and run;

`docker build --file ./Dockerfile -t <your DockerHub username>/<a name for this image>:<a tag, where leaving this blank applies "latest" which may be fine for now> .`

Mine looks like this;

`docker build --file ./Dockerfile -t freshdemo/juice-shop-dockerhub-gke:new .`

## Step 4 - Verify the Image

If all goes well you can run the following command to list your images;

`docker images`

`freshdemo/juice-shop-dockerhub-gke new ca790e02106c 38 hours ago 632MB`

## Step 5 - Push the Image to the Container Registry

To push the image to the registry run;

`docker push <your DockerHub username>/<a name for this image>:<a tag, where leaving this blank applies "latest" which may be fine for now>`

Mine looks like this;

`docker push freshdemo/juice-shop-dockerhub-gke:new`

### Optional 1 - Delete the Image

To delete the image run;

`docker rmi <image name or ID>`

### Optional 2 - Run a Container from the Image

This step is going to vary between containers. In this documentation you will find a step in the associated code repository post so search for the tags `ninjastars` and `coderepo`.
