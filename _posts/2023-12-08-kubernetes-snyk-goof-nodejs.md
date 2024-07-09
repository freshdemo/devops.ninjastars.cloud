---
layout: post
title:  "Kubernetes - Goof Node.js"
date:   2023-12-08 21:27:07 -0400
categories: infrastructure kubernetes goof node.js
---

Running an application in a Kubernetes environment allows you to demonstrate the Snyk Kubernetes integrations.

## Step 1 - Build a Kubernetes Cluster

You can build this on an x86 based laptop or other systems you may have laying around. Deploying k8s on an M1 Mac and Raspberry Pi’s has been largely ineffective and a waste of time.

Alternatively you can use this guide to deploy in our Google Cloud environment, [GKE Kubernetes Cluster](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/10/20/1719238881).

## Step 2 - Build a Container

This guide outlines how to build a container that will run on x86 hardware, [Containers - Snyk Goof Node.js](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/12/05/1768358050).

## Step 3 - Create a Namespace

Creating a dedicated namespace in k8s allows you to easily create and destroy deployments without worrying about wrecking other working apps. You are able to skip this step, and everything created will be deployed to the `default` namespace.

Use a naming convention that works for you. Because we often will use the same code in multiple SCM’s, Container Registries, and CI/CD tools that is the convention below. I.e. the app is the Node.js version of goof, the code is hosted in GitHub repos, the container is stored in DockerHub, and the container will be run in Google Kubernetes Engine:

`kubectl create namespace node-goof-github-dockerhub-gke`

## Step 4 - Develop a Kubernetes Deployment for MondoDB

This application requires a functional mongoDB instance. Create a file called `goof-mongo-deployment.yml` and copy/paste the script below into it. Some notes on the configuration file:

-   Keeping the same namespace is important to ensure connectivity.
    
-   In the `spec` section there is an environment variable, `DOCKER = 1`. This is critical in getting the pods to communicate.
    
-   The image is specifically using `mongo:3`. Other versions don't seem to work.
    

`apiVersion: apps/v1`  
`kind: Deployment`  
`metadata:`  
`annotations:`  
`kompose.cmd: kompose convert`  
`kompose.version: 1.31.2 (HEAD)`  
`creationTimestamp: null`  
`labels:`  
`app: node-goof-mongo-github-dockerhub-gke`  
`io.kompose.service: goof-mongo`  
`name: node-goof-mongo-github-dockerhub-gke`  
`namespace: node-goof-github-dockerhub-gke`  
`spec:`  
`replicas: 1`  
`selector:`  
`matchLabels:`  
`app: node-goof-mongo-github-dockerhub-gke`  
`strategy: {}`  
`template:`  
`metadata:`  
`annotations:`  
`kompose.cmd: kompose convert`  
`kompose.version: 1.31.2 (HEAD)`  
`creationTimestamp: null`  
`labels:`  
`app: node-goof-mongo-github-dockerhub-gke`  
`io.kompose.network/node-goof-github-dockerhub-gke-default: "true"`  
`io.kompose.service: node-goof-mongo-github-dockerhub-gke`  
`spec:`  
`containers:`  
`- env:`  
`- name: DOCKER`  
`value: "1"`  
`image: mongo:3`  
`name: node-goof-mongo-github-dockerhub-gke`  
`ports:`  
`- containerPort: 27017`  
`hostPort: 27017`  
`protocol: TCP`  
`resources: {}`  
`restartPolicy: Always`  
`status: {}`

## Step 4 - Apply the MongoDB Configuration

Deploy the MongoDB configuration above with the following:

`kubectl apply -f ./goof-mongo-deployment.yml`

Later when you want to remove the configuration:

`kubectl delete -f ./goof-mongo-deployment.yml`

## Step 4 - Get the MongoDB IP

We’ll need to tell the app server where the DB is. Running the following command will provide the IP address of the MongoDB pod. Be sure to replace the namespace (-n) with whatever you’ve used.

`kubectl get pods -n node-goof-github-dockerhub-gke -o wide`

We’ll be using this data in the next steps.

## Step 5 - Develop a Kubernetes Deployment for Goof

Create a deployment and service for the app server.

Create a file called `goof-deployment.yml` with the following contents. Some notes on the configuration file:

-   The namespace must be the same in all of these files.
    
-   In the `spec` section there is an environment variable, `DOCKER = 1`. This is critical in getting the pods to communicate.
    
-   Ensure the `image` being used is the correct one for x86/amd systems and not a container that runs on your laptop.
    
-   The `ip` under `hostAliases` is the IP address of the MongoDB server identified in the previous step.
    

`apiVersion: apps/v1`  
`kind: Deployment`  
`metadata:`  
`annotations:`  
`kompose.cmd: kompose convert`  
`kompose.version: 1.31.2 (HEAD)`  
`creationTimestamp: null`  
`labels:`  
`app: node-goof-github-dockerhub-gke`  
`io.kompose.service: node-goof-github-dockerhub-gke`  
`name: node-goof-github-dockerhub-gke`  
`namespace: node-goof-github-dockerhub-gke`  
`spec:`  
`replicas: 1`  
`selector:`  
`matchLabels:`  
`app: node-goof-github-dockerhub-gke`  
`strategy: {}`  
`template:`  
`metadata:`  
`labels:`  
`app: node-goof-github-dockerhub-gke`  
`io.kompose.service: node-goof-github-dockerhub-gke`  
`spec:`  
`containers:`  
`- env:`  
`- name: DOCKER`  
`value: "1"`  
`image: freshdemo/node-goof-github-dockerhub-gke:gha@sha256:8941c76ce8908b6e1ed04ef58ac5f7981875d7ed7b45730bbbfdbc73f5208410`  
`name: node-goof-github-dockerhub-gke`  
`resources: {}`  
`restartPolicy: Always`  
`hostAliases:`  
`- ip: "10.124.2.54"`  
`hostnames:`  
`- "goof-mongo"`  
`status: {}`

## Step 6 - Develop a Kubernetes Service for Goof

Create a file called `goof-service.yml` with the following contents. You could also include this configuration in the deployment file to consolidate, however it’s sometimes handy to leave the service intact while you delete and reapply the deployment. Some notes on the configuration file:

-   The namespace must match across all of these files.
    
-   The `port` 3001 can be modified, however leave the `targetPort` as 3001. `port` is what you'll use to access the service.
    

`apiVersion: v1`  
`kind: Service`  
`metadata:`  
`labels:`  
`app: node-goof-github-dockerhub-gke`  
`name: node-goof-github-dockerhub-gke`  
`namespace: node-goof-github-dockerhub-gke`  
`spec:`  
`type: LoadBalancer`  
`ports:`  
`- name: "3001"`  
`port: 3001`  
`targetPort: 3001`  
`- name: "9229"`  
`port: 9229`  
`targetPort: 9229`  
`selector:`  
`app: node-goof-github-dockerhub-gke`

## Step 7 - Apply the Kubernetes Deployment & Service for Goof

You can apply both configurations at the same time with the following:

`kubectl apply -f ./goof-deployment.yml,goof-service.yml`

Make sure the pods have been created successfully with:

`kubectl get pods -n node-goof-github-dockerhub-gke -o wide`

## Step 8 - Access the Application

To get to the application we need to determine the public IP address of the service.

`kubectl get services -n node-goof-github-dockerhub-gke`

From here you can access the application by visiting `http://<service IP>:3001`

## Step 9 - Destroy the Configurations

These are vulnerable applications and should not be left running. You can either delete the various parts by specifying individual configuration files, or use the following to remove everything;

`kubectl delete -f ./goof-deployment.yml,goof-service.yml,goof-mongo-deployment.yml`

## Troubleshooting

### Troubleshooting

1.  Check to see what’s running in your namespace
    
    1.  `kubectl get pods -n node-goof-github-dockerhub-gke -o wide`
        
2.  Check the logs of the pod identified in step 1 above. This can be helpful to identify issues with your container or the app not starting.
    
    1.  `kubectl logs -n node-goof-github-dockerhub-gke node-goof-github-dockerhub-gke-648cc55d8d-s6qcp`
        
3.  Check the services to get the public IP address
    
    1.  `kubectl get services -n node-goof-github-dockerhub-gke -o wide`
        
4.  Delete the pod identified in step 1. This can be helpful if you want to re-pull the container image.
    
    1.  `kubectl delete pod -n node-goof-github-dockerhub-gke node-goof-github-dockerhub-gke-648cc55d8d-s6qcp`
        
5.  Make sure the environment variable DOCKER = 1 is set correctly in the MongoDB and Goof deployment, otherwise they will not be able to communicate and the app will not start.
    

Be the first to add a reaction