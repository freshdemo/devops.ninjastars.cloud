---
layout: post
title:  "Kubernetes - Juice Shop"
date:   2023-12-08 21:27:07 -0400
tags: infrastructure kubernetes juice-shop
---

Running an application in a Kubernetes environment allows you to demonstrate the Snyk Kubernetes integrations.

## Step 1 - Build a Kubernetes Cluster

You can build this on an x86 based laptop or other systems you may have laying around. Deploying k8s on an M1 Mac and Raspberry Pi’s has been largely ineffective and a waste of time.

Alternatively you can use this guide to deploy in our Google Cloud environment, [GKE Kubernetes Cluster](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/10/20/1719238881).

## Step 2 - Build a Container

This guide outlines how to build a container that will run on x86 hardware,

[Containers - Juice Shop Node.js](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/30/1768915028).

## Step 3 - Create a Namespace

Creating a dedicated namespace in k8s allows you to easily create and destroy deployments without worrying about wrecking other working apps. You are able to skip this step, and everything created will be deployed to the `default` namespace.

Use a naming convention that works for you. Because we often will use the same code in multiple SCM’s, Container Registries, and CI/CD tools that is the convention below. I.e. the app is the java version of goof, the code is hosted in Azure repos, the container is stored in Google Artifact Registry, and the container will be run in Google Kubernetes Engine:

`kubectl create namespace juice-shop-github-dockerhub-gke`

## Step 4 - Develop a Kubernetes Deployment & Service

Most of the demonstration repos in this blog have a k8s directory with similar configurations in them. Some notes on the configuration file:

-   Ensure the image is the one you built for AMD/x86 using the --platform option.
    
-   The `port 8082` can be modified and is what you will be accessing. Leave the `targetPort` alone.
    
-   The `---` between the deployment and service is required.
    

Create a file called `juice-shop-deployment.yml`

`apiVersion: apps/v1`  
`kind: Deployment`  
`metadata:`  
`name: juice-shop-github-dockerhub-gke`  
`labels:`  
`app: juice-shop-github-dockerhub-gke`  
`spec:`  
`selector:`  
`matchLabels:`  
`app: juice-shop-github-dockerhub-gke`  
`replicas: 1`  
`template:`  
`metadata:`  
`labels:`  
`app: juice-shop-github-dockerhub-gke`  
`spec:`  
`containers:`  
`- name: juice-shop-github-dockerhub-gke`  
`image: freshdemo/juice-shop-github-dockerhub-gke:gha`  
`# imagePullPolicy: IfNotPresent`  
`imagePullPolicy: Always`  
`ports:`  
`- containerPort: 3000`  
`securityContext:`  
`privileged: true`

\---

`apiVersion: v1`  
`kind: Service`  
`metadata:`  
`labels:`  
`app: juice-shop-github-dockerhub-gke`  
`name: juice-shop-github-dockerhub-gke`  
`spec:`  
`type: LoadBalancer`  
`ports:`  
`- port: 8082`  
`protocol: TCP`  
`targetPort: 3000`  
`selector:`  
`app: juice-shop-github-dockerhub-gke`

## Step 5 - Apply the Kubernetes Configuration

The script above has both the deployment of the container, and the services to access its ports in one file. To build this in your k8s environment run:

`kubernetes apply -f juice-shop-deployment.yml`

## Step 6 - Check Your Work

1.  Check to see what’s running in your namespace
    
    1.  `kubectl get pods -n juice-shop-github-dockerhub-gke -o wide`
        
2.  Check the services to get the public IP address
    
    1.  `kubectl get services -n juice-shop-github-dockerhub-gke -o wide`
        
3.  Check to see if the app is running properly.
    
    1.  From the previous command copy the `EXTERNAL-IP` and port on the left side of the :, and navigate to that in a browser (i.e. `35.203.27.216:8082`).
        

## Step 7 - Delete the Kubernetes Configuration

Once you are done with this app you can delete the configuration;

`kubernetes delete -f juice-shop-deployment.yml`

### Troubleshooting

1.  Check to see what’s running in your namespace
    
    1.  `kubectl get pods -n juice-shop-github-dockerhub-gke -o wide`
        
2.  Check the logs of the pod identified in step 1 above. This can be helpful to identify issues with your container or the app not starting.
    
    1.  `kubectl logs -n juice-shop-github-dockerhub-gke juice-shop-github-dockerhub-gke-648cc55d8d-s6qcp`
        
3.  Check the services to get the public IP address
    
    1.  `kubectl get services -n juice-shop-github-dockerhub-gke -o wide`
        
4.  Delete the pod identified in step 1. This can be helpful if you want to re-pull the container image.
    
    1.  `kubectl delete pod -n juice-shop-github-dockerhub-gke juice-shop-github-dockerhub-gke-648cc55d8d-s6qcp`
        
