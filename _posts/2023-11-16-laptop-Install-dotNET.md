---
layout: post
title:  "Laptop - Install .NET"
date:   2023-11-16 21:27:07 -0400
tags: infrastructure dotnet
---

Getting your developer environment setup is critical for demos. .NET is popular in large and small enterprises, and there are lots of legacy applications.

## Step 1 - Install the .Net SDK

Download the ARM64 version from here, [Download .NET 8.0 (Linux, macOS, and Windows)](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) . Take note of the directory being created, you can put this anywhere on your system.

`mkdir -p $HOME/dotnet`

`tar zxf dotnet-sdk-8.0.100-osx-arm64.tar.gz -C $HOME/dotnet`

## Step 2 - Install the ASP.NET Core Runtime

This let’s you run apps in a webserver.

`tar zxf aspnetcore-runtime-8.0.0-osx-arm64.tar.gz -C $HOME/dotnet`

## Step 3 - Install the .NET Runtime

This lets you run console apps.

`tar zxf dotnet-runtime-8.0.0-osx-arm64.tar.gz -C $HOME/dotnet`

## Step 4 - Setup System Environment Variables

Tell your system where to access .NET by adding the following lines to your ~/.zshrc file. You can also enter these on the terminal to getting it working, but new terminal windows will not have the settings;

`export DOTNET_ROOT=$HOME/dotnet`  
`export PATH=$PATH:$HOME/dotnet`

You can test the installation with;

`dotnet --version`

Be the first to add a reaction