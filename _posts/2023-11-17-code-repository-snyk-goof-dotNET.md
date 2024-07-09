---
layout: post
title:  "Code Repository - Snyk Goof .NET"
date:   2023-11-16 21:27:07 -0400
categories: code repository dotnet goof
---

This isn’t actually the same Goof we know and love, but it is a .NET app that should do the trick.

## Step 1 - Get ASP & .NET

This repo actually calls for a much older version of .NET, however you can install version 8 by following these steps, [Laptop Install .NET](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1750630420).

## Step 2 - Get Snyk Goof .NET

The original repo can be found here, [![](Code%20Repository%20-%20Snyk%20Goof%20.NET%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub - snyk-matt/dotNET-goof-v2](https://github.com/snyk-matt/dotNET-goof-v2), however this Matt is no longer with Snyk so I’ve created a clone of it here, [![](Code%20Repository%20-%20Snyk%20Goof%20.NET%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub - stephen-snyk/goof-dotnet](https://github.com/stephen-snyk/goof-dotnet).

`git clone https://github.com/stephen-snyk/goof-dotnet.git`

## Step 3 - Build the App

This app was built long ago with an already dated version of .NET. To upgrade the version and get it to build correctly install the .NET upgrade assistant [How to install the .NET Upgrade Assistant - .NET Core](https://learn.microsoft.com/en-us/dotnet/core/porting/upgrade-assistant-install#install-the-net-global-tool), and run it with;

`dotnet tool install -g upgrade-assistant`

`~/.dotnet/tools/upgrade-assistant upgrade`

cd into the directory you cloned the repo, then run;

`dotnet build`

### Optional 1 - Run the App

If you want to see the application running;

From the repo directory change into the sub directory;

`cd dotNETGoofV2.Website`

Run the app with;

`dotnet run`

Then browse to `http://localhost:5000`

### Optional 2 - Build a Container & Push to Registry

There does not appear to be a built container on Docker Hub for this project.

See this guide on details to build your own container and push it to your own Container Registry, [Build Containers & Push to Registry](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1752432748).

Be the first to add a reaction