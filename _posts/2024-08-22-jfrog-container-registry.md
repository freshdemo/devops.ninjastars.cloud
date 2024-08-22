---
layout: post
title:  "JFrog Artifactory - Container Registry"
date:   2024-06-20 21:27:07 -0400
tags: infrastructure container jfrog
---

There are 3 benefits to integrating JFrog Artifactory:

1.  Package Repository integration that will help Snyk identify the package information in Artifactory when the pom.xml indicates the location of the private registry (in the case where packages won’t be verified from the public sources).
    
2.  Gatekeeper to block people from downloading open source packages based on severity of vulnerabilities identified. NOTE as of the time of this writing Plugins are not supported in the Open Source version of Artifactory so Gatekeeper is not something you’ll be able to demonstrate from your own environment [![](https://jfrog.com/help/favicon.ico)JFrog Help Center](https://jfrog.com/help/r/jfrog-hosting-models-documentation/artifactory-comparison-matrix) .
    
3.  On-premise Container Registry integration, where Snyk can find container images to scan.
    

This post will focus on deploying JFrog Container Registry Container.

Instructions directly from JFrog are here, [![](https://jfrog.com/help/favicon.ico)JFrog Help Center](https://jfrog.com/help/r/jfrog-installation-setup-documentation/install-artifactory-single-node-with-docker?section=UUID-999fbbd5-ed0b-361e-e622-5bf2bc159060_UUID-6560a094-94c2-ca03-359f-ccb55be0e480) .

## Step 1 - Docker

You will need Docker Desktop installed on the host system to demonstrate this. Here is a post to set it up if you haven’t already, [![](https://docs.docker.com/favicons/docs@2x.ico)Install Docker Desktop on Mac](https://docs.docker.com/desktop/install/mac-install/).

## Step 2 - Environment Setup

If you’ve already setup JFrog Artifact Repository you can skip the following steps, [JFrog Artifactory Package Repository](https://snyksec.atlassian.net/wiki/spaces/FBK/pages/2180874589).

-   Create a `$JFROG_HOME` directory somewhere on your system. I use `mkdir ~/Documents/jfrog`
    
-   Setup an environment variable for `$JFROG_HOME` so that it can be found. Add `export JFROG_HOME=~/Documents/jfrog` to your ~/.zshrc, save the file.
    
-   Type `source ~/.zshrc` to load the new settings into your current terminal session.
    

As per the documents on the JFrog website run;

`mkdir -p $JFROG_HOME/artifactory/var/etc/ cd $JFROG_HOME/artifactory/var/etc/ touch ./system.yaml sudo chown -R 1030:1030 $JFROG_HOME/artifactory/var sudo chmod -R 777 $JFROG_HOME/artifactory/var`

## Step 3 - Build a Container

A couple of notes:

-   Make sure to stop any other containers running with the ports in this run command. JFrog did not like when I changed the ports and would not run correctly.
    
-   The Artifactory also did not run when it was instantiated while the host system was connected to a VPN.
    
-   If you’ve already setup JFrog Artifact Repository the credentials will be the same as Artifactory, [JFrog Artifactory Package Repository](https://snyksec.atlassian.net/wiki/spaces/FBK/pages/2180874589).
    

`docker run --name artifactory -v $JFROG_HOME/artifactory/var/:/var/opt/jfrog/artifactory -d -p 8081:8081 -p 8082:8082 releases-docker.jfrog.io/jfrog/artifactory-jcr:latest`

Once completed you can navigate to `http://localhost:8082/ui/` and login with admin/password.

## Step 4 - Configure Repositories

Create a local Docker repository (like your own Docker Hub):

-   Navigate to `Administration (gears top left), Repositories, Repositories`. Click `Add Repositories, Local Repositories`.
    
-   Choose `Docker` as the type.
    
-   For the `Repository Key` type `Docker`.
    
-   Click `Create`.
    

Configure the way to interact with the repository:

-   For `Docker Access Method` choose `Repository Path`.
    
-   For `Service Provider` choose `NGINX`.
    
-   `Internal Hostname` and `Public Server Name` should both be set to the internal IP address of your laptop. If this isn’t currently an option navigate to `Administration (gears top left)`, `Authentication Providers, General`, and set the `Custom Base URL` to `http://<your internal IP>:8082`.
    

## Step 5 - Prepare Your Laptop to Push Images

Docker defaults to push over TLS only.

-   Modify or add the file from the command line with `vi ~/.docker/daemon.json`
    
-   Add the insecure-registries stanza similar to the one below, modifying the 192.168.1.76 to match your systems internal IP address.
    

`{ "builder": { "gc": { "defaultKeepStorage": "20GB", "enabled": true } }, "experimental": false, "insecure-registries": [ "192.168.1.76:8082" ] }`

-   Restart Docker Desktop.
    
-   Login to your Container Registry with `docker login <your IP>:8082`. You should see a success message. If this step fails revisit Step 4 and the previous steps in this section as they are critical to making this work.
    

## Step 6 - Push an Image

For the sake of this test we will pick a small container image, re-tag it, and push it to the registry to prove the previous steps were implemented successfully.

-   From the CLI run `docker pull busybox:latest`.
    
-   Find the image ID of your new busybox image with `docker images`.
    
-   Tag your busybox image with the name of your Container Registry `docker tag 3fba0c87fcc8 192.168.1.76:8082/docker/busybox:latest`. Replace `3fba0c87fcc8` with the image ID you got in step 2, and `192.168.1.76` is your internal IP.
    
-   Push the image to the Container Registry `docker push 192.168.1.76:8082/docker/busyboy:latest`. Again replacing the IP address with your own.
    
-   The IP address, port, and path are critical. If the push step fails go back and review previous steps.
    

## Step 7 - Enable SSL

At the time of this writing Broker requires downstream Container Registries to be https and not http. I found this out by searching support tickets.

The instructions from JFrog worked for me, [![](https://jfrog.com/help/favicon.ico)JFrog Help Center](https://jfrog.com/help/r/artifactory-how-to-enable-tls-within-the-jfrog-platform-using-custom-certificates/steps-to-enable-tls-with-customer-certificate).

## Step 8 - Broker

The Container Registry integration requires two Broker containers; The Broker client container specific to container registries, and the Container Registry agent.

### Broker Client

After much trial and error this is the minimum viable configuration identified.

-   Setup the Organization to for Container Registry Broker support.
    
    -   From [https://app.snyk.io/admin](https://app.snyk.io/admin "https://app.snyk.io/admin") navigate to the Organization.
        
    -   Add Feature Flags for `artifactory` and `artifactoryBroker`.
        
    -   Make sure the Entitlements `artifactoryCr`, `containerRegistryIntegrations`, and `customRegistries` are enabled.
        
    -   Create a new Broker connection for `artifactory-cr`, and save the token.
        
-   `CR_AGENT_URL` you can change the port to whatever works for your system so long as it doesn’t conflict with another app on your system, and just remember to use this port when setting up the Container Registry agent.
    
-   `CR_TYPE` is specific so ensure you use `artifactory-cr` as is.
    

`docker run --restart=always \ -p 8000:8000 \ -e BROKER_TOKEN="<broker token>" \ -e BROKER_CLIENT_URL="http://192.168.1.76:8000" \ -e CR_AGENT_URL="http://192.168.1.76:8083" \ -e CR_TYPE="artifactory-cr" \ -e CR_BASE="192.168.1.76:8082/artifactory/api/docker/docker" \ -e CR_USERNAME="admin" \ -e CR_PASSWORD="<artifactory password>" \ -e PORT=8000 \ snyk/broker:container-registry-agent`

### Container Registry Agent

-   The port must match what you set CR\_AGENT\_URL to in the Broker setup above.
    
-   I needed both NODE\_TLS\_REJECT\_UNAUTHORIZED and INSECURE\_DOWNSTREAM variables set, otherwise there were ssl errors.
    

`docker run --restart=always \ -p 8083:8083 \ -e SNYK_PORT=8083 \ -e NODE_TLS_REJECT_UNAUTHORIZED=0 \ -e INSECURE_DOWNSTREAM="true" \ snyk/container-registry-agent:latest`

## Step 9 - Import an Image

-   Navigate back to [https://app.snyk.io](https://app.snyk.io/ "https://app.snyk.io"), and to the Organization that has Artifactory and Broker enabled.
    
-   Click Add Project, and choose Artifactory, then select your image.
    
    -   If this works you’ll have an empty project since busybox doesn’t typically have any issues.
        
    -   If this doesn’t work loop back and troubleshoot the both Broker container logs.
        

## Step 10 - Optionally Use a Vulnerable Image

-   There are many documents on how to create specific containers for Juice Shop and Goof and push them to registries, [https://snyksec.atlassian.net/wiki/label/~629db3cb76c0360069f263e7/container](https://snyksec.atlassian.net/wiki/label/~629db3cb76c0360069f263e7/container "https://snyksec.atlassian.net/wiki/label/~629db3cb76c0360069f263e7/container").
    
-   The main difference to push them to your Artifactory Container Registry is to tag the image with your Container Registry so that docker push knows where to send it. Step 6 of this document covers this [JFrog Container Registry Container & Integration | Step 6 Push an Image](https://snyksec.atlassian.net/wiki/spaces/FBK/pages/edit-v2/2198175783?draftShareId=a4d79f5d-4927-4cf4-a594-f7eb0ac368d3#Step-6---Push-an-Image) .
    

Be the first to add a reaction