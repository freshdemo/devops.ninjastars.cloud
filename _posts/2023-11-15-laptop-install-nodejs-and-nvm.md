---
layout: post
title:  "Laptop - Install Node.js & NVM"
date:   2023-11-15 21:27:07 -0400
tags: infrastructure node.js nvm
---
Getting your developer environment setup is critical for demos. Node Version Manager helps you quickly install various versions of Node.js, and select which one to use during runtime. This is particularly useful when you have different Node.js projects on your laptop.

## Step 1 - Install nvm

Use brew to install locally.

`brew update`

`brew install nvm`

## Step 2 - Install versions of Node.js

You are able to request a major or specific version. For example both of these are acceptable and depend on your use case;

`nvm install v18`

`nvm install v18.17.1`

## Step 3 - List versions available

To see what’s installed on your system use;

`nvm list`

## Step 4 - Select a version of Node.js

From the list in the previous command select the version to use;

`nvm use v18.17.1`
