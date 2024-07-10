---
layout: post
title:  "Code Repository - Juice Shop"
date:   2023-11-16 21:27:07 -0400
tags: code repository juice-shop node.js
---
Juice Shop has the advantage of being an OWASP project, making it more agnostic than using one of Snyk’s vulnerable projects. It’s JavaScript and well maintained making it pretty modern. You can get the source directly from here, [![](Code%20Repository%20-%20Juice%20Shop%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)OWASP Juice Shop](https://github.com/juice-shop) , but below you will find some specifics on getting it working in your environment.

**For a start-to-end demo using this app it is recommended to use GitHub Actions, particularly for building a container that will run in Kubernetes.** On M1 Mac’s Juice Shop you are not able to build a cross platform x86 architecture docker container. This is problematic if you wanted to have a full pipeline from code, to container, to Kubernetes or similar. GitHub Actions allows you to build in their cloud, rather than using a local runner like most of the other CI tools, so this is a good option if you’re looking for one.

## Step 1 - Get Node.js & npm

Getting the right version of build tools for the application is critical. At the time of writing, `Node.js version 18` builds well for me in all environments.

This post can help you get Node.js deployed, [Laptop Install Node.js and nvm](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/15/1749483611) .

## Step 2 - Get Juice Shop

You can get the latest code directly from their repo here, [![](Code%20Repository%20-%20Juice%20Shop%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub - juice-shop/juice-shop: OWASP Juice Shop: Probably the most modern and sophisticated insecure web application](https://github.com/juice-shop/juice-shop) . Alternatively you can get it from my repo which includes a few modifications, [![](Code%20Repository%20-%20Juice%20Shop%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub - stephen-snyk/juice-shop-github-dockerhub-gke](https://github.com/stephen-snyk/juice-shop-github-dockerhub-gke) .

Change to the directory you want to put the code and clone the repo;

`git clone https://github.com/juice-shop/juice-shop.git`

## Step 3 - Build the App

cd into the directory you cloned the repo, then run;

`npm install`

You should see it downloading all of the open source software dependencies.

If you’re just trying to scan the app with the CLI, IDE, or CI integrations this is all you need to do.

### Optional 1 - Run the App

If you want to see the application running;

`npm start`

Then browse to `http://localhost:3000`

### Optional 2 - Build a Container & Push to Registry

The developer has a container that you can pull from [https://hub.docker.com/r/bkimminich/juice-shop](https://hub.docker.com/r/bkimminich/juice-shop) .

See this guide on details to build your own container and push it to your own Container Registry, [Build Containers & Push to Registry](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1752432748) .

To reiterate the statement above, building a container for this app seems to work fine locally on an M1 Mac. Even when building with --platform=linux/x86,linuxamd64 the container does not run in GKE. As a result I’ve resorted to hosting the code on GitHub and using GitHub Actions to build the container and push it to the registry.
