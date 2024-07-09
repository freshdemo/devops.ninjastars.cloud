---
layout: post
title:  "Kubernetes - Snyk Goof Java"
date:   2023-12-08 21:27:07 -0400
categories: infrastructure kubernetes goof
---

Running an application in a Kubernetes environment allows you to demonstrate the Snyk Kubernetes integrations.

## Step 1 - Build a Kubernetes Cluster

You can build this on an x86 based laptop or other systems you may have laying around. Deploying k8s on an M1 Mac and Raspberry Pi’s has been largely ineffective and a waste of time.

Alternatively you can use this guide to deploy in our Google Cloud environment, [GKE Kubernetes Cluster](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/10/20/1719238881).

## Step 2 - Build a Container

This guide outlines how to build a container that will run on x86 hardware, [Containers - Snyk Goof Java](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/30/1768980481).

## Step 3 - Create a Namespace

Creating a dedicated namespace in k8s allows you to easily create and destroy deployments without worrying about wrecking other working apps. You are able to skip this step, and everything created will be deployed to the `default` namespace.

Use a naming convention that works for you. Because we often will use the same code in multiple SCM’s, Container Registries, and CI/CD tools that is the convention below. I.e. the app is the java version of goof, the code is hosted in Azure repos, the container is stored in Google Artifact Registry, and the container will be run in Google Kubernetes Engine:

`kubectl create namespace java-goof-azure-gar-gke`

## Step 4 - Develop a Kubernetes Deployment & Service

Most of the demonstration repos in this blog have a k8s directory with similar configurations in them. Some notes on the configuration file:

-   The app, name, and namespace are important because they are used to stitch a service to the deployment and make sure they are in the same namespace.
    
-   Under containers, image is where you’ll set the container to pull. Make sure the container build aligns with the architecture your Kubernetes is running on (i.e. If it's on your laptop it must be ARM64, if it’s in GKE it is AMD64).
    
-   Under loadbalancer port, the first port is what will be advertised to the world so this can be anything but you need to ensure that perimeter firewalls are permitting traffic from your IP address to that port. The targetport is what the container is actually listening on, so within these blogs don’t change that unless you’ve gone into the source code and modified the port prior to building the container.
    
-   Save this file as `java-goof-deployment.yml` or something similar.
    

`apiVersion: apps/v1`  
`kind: Deployment`  
`metadata:`  
`labels:`  
`app: java-goof-azure-gar-gke`  
`name: java-goof-azure-gar-gke`  
`namespace: java-goof-azure-gar-gke`  
`spec:`  
`replicas: 1`  
`selector:`  
`matchLabels:`  
`app: java-goof-azure-gar-gke`  
`template:`  
`metadata:`  
`labels:`  
`app: java-goof-azure-gar-gke`  
`spec:`  
`containers:`  
`- image: northamerica-northeast2-docker.pkg.dev/snyk-cx-se-demo/sperciballi-registry/java-goof-azure-gar-gke:gke`  
`imagePullPolicy: Always`  
`name: java-goof-azure-gar-gke`  
`restartPolicy: Always`

`apiVersion: v1`  
`kind: Service`  
`metadata:`  
`labels:`  
`app: java-goof-azure-gar-gke`  
`name: java-goof-azure-gar-gke`  
`namespace: java-goof-azure-gar-gke`  
`spec:`  
`type: LoadBalancer`  
`ports:`  
`- port: 8082`  
`protocol: TCP`  
`targetPort: 8080`  
`selector:`  
`app: java-goof-azure-gar-gke`

## Step 5 - Apply the Kubernetes Configuration

The script above has both the deployment of the container, and the services to access its ports in one file. To build this in your k8s environment run:

`kubernetes apply -f <path to the yml file>`

## Step 6 - Check Your Work

1.  Check to see what’s running in your namespace
    
    1.  `kubectl get pods -n java-goof-azure-gar-gke -o wide`
        
2.  Check the services to get the public IP address
    
    1.  `kubectl get services -n java-goof-azure-gar-gke -o wide`
        
3.  Check to see if the app is running properly.
    
    1.  From the previous command copy the `EXTERNAL-IP` and port on the left side of the :, and navigate to that in a browser (i.e. `35.203.27.216:8082`). You can also add `/todolist` after the port to see that specific app.
        

## Step 7 - Delete the Kubernetes Configuration

Once you are done with this app you can delete the configuration;

`kubernetes delete -f <path to the yml file>`

### Troubleshooting

1.  Check to see what’s running in your namespace
    
    1.  `kubectl get pods -n java-goof-azure-gar-gke -o wide`
        
2.  Check the logs of the pod identified in step 1 above. This can be helpful to identify issues with your container or the app not starting.
    
    1.  `kubectl logs -n java-goof-azure-gar-gke java-goof-azure-gar-gke-648cc55d8d-s6qcp`
        
3.  Check the services to get the public IP address
    
    1.  `kubectl get services -n java-goof-azure-gar-gke -o wide`
        
4.  Delete the pod identified in step 1. This can be helpful if you want to re-pull the container image.
    
    1.  `kubectl delete pod -n java-goof-azure-gar-gke java-goof-azure-gar-gke-648cc55d8d-s6qcp`
        

Be the first to add a reaction