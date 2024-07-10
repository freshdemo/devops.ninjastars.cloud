---
layout: post
title:  "Code Repository - Snyk Goof Node.js"
date:   2023-12-01 21:27:07 -0400
tags: code repository goof node.js
---

Goof has oodles of vulnerabilities, builds quickly, and the app works.

## Step 1 - Get Node.js & npm

Getting the right version of build tools for the application is critical. This version will build on most Node.js versions, **however if you want it to run properly you’ll need to use** `nvm install v18` and `nvm use v18.17.1` while following the guide below. None of the newer versions of Node.js work.

This post can help you get Node.js deployed, [Laptop Install Node.js and nvm](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/15/1749483611).

## Step 2 - Get Snyk Goof Node.js

You can get the latest code directly from their repo here,[![](Code%20Repository%20-%20Snyk%20Goof%20Node.js%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub - snyk-labs/nodejs-goof: Super vulnerable todo list application](https://github.com/snyk-labs/nodejs-goof).

Change to the directory you want to put the code and clone the repo;

`git clone https://github.com/snyk-labs/nodejs-goof.git`

## Step 3 - Build the App

cd into the directory you cloned the repo, then run;

`npm install`

You should see it downloading all of the open source software dependencies.

If you’re just trying to scan the app with the CLI, IDE, or CI integrations this is all you need to do.

### Optional 1 - Run the App

If you want to see the application running, start by deploying the required mongoDB. This command will deploy mongoDB in a container that gets destroyed (--rm) once you terminate the command (do this from another terminal window);

`docker run --rm -p 27017:27017 mongo:3`

Then, from the project root directory;

`npm start`

Then browse to `http://localhost:3001`

### Optional 2 - Build a Container & Push to Registry

The developer has a container that you can pull from [https://hub.docker.com/r/snyklabs/goof](https://hub.docker.com/r/snyklabs/goof).

See this guide on details to build your own container and push it to your own Container Registry, [https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1752432748](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1752432748).

Be the first to add a reaction