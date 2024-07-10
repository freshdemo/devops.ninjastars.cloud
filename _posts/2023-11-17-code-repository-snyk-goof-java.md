---
layout: post
title:  "Code Repository - Snyk Goof Java"
date:   2023-11-16 21:27:07 -0400
tags: code repository java goof
---

Goof has oodles of vulnerabilities, builds quickly, and the app works.

## Step 1 - Get Java & Maven

The Goof repo states that you need JDK 8 for this to work. JDK 8 does not have a Mac arm64 distribution. The good news is that it works with JDK 11, which you can download here [![](Code%20Repository%20-%20Snyk%20Goof%20Java%20-%20Stephen%20Perciballi%20-%20Confluence/favicon-32.png)Java Archive Downloads - Java SE 11](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html) .

When doing this on your laptop follow here, [Laptop Install Java & maven](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/15/1750204429) .

Make sure to set the environment variables on your laptop (in the instructions above) to use JDK 11.

## Step 2 - Get Snyk Goof Java

Get the latest version from here, [![](Code%20Repository%20-%20Snyk%20Goof%20Java%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub - snyk-labs/java-goof](https://github.com/snyk-labs/java-goof) .

Change to the directory you want to put the code and clone the repo;

`git clone https://github.com/snyk-labs/java-goof.git`

## Step 3 - Build the App

cd into the directory you cloned the repo, then run;

`mvn install`

### Optional 1 - Run the App

If you want to see the application running;

`mvn tomcat7:run`

Then browse to `http://localhost:8080/`

See the source repository README file for other fun things you can do with this app, like exploitation.

### Optional 2 - Build a Container & Push to Registry

There does not appear to be a built container on Docker Hub for the Java version of this project.

See this guide on details to build your own container and push it to your own Container Registry, [Build Containers & Push to Registry](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1752432748). There are technically two apps in this project so make sure to `cd todolist-goof` prior to building the container.
