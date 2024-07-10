---
layout: post
title:  "GKE Kubernetes Cluster"
date:   2023-10-20 21:27:07 -0400
tags: kubernetes infrastructure
---
It’s a good idea to at least try to setup your own Kubernetes cluster, deploy an application to it, and integrate with Snyk. It’s not a good idea to leave apps running, or even the cluster. This guide is designed to get your cluster running in Google Kubernetes Engine quickly enough that you feel comfortable deleting it when done using it.

**Wherever possible include your Okta username in the resources you create in GKE so that our IT security team can reach you quickly if required.**

## Step 1

Get the gcloud SDK, [![](GKE%20Kubernetes%20Cluster%20-%20Stephen%20Perciballi%20-%20Confluence/favicon.ico)Install the gcloud CLI  |  Google Cloud CLI Documentation](https://cloud.google.com/sdk/docs/install) .

Once installed you should be able to move onto the next steps after the following command runs successfully:

`gcloud init`

This should walk you through authenticating with your Okta account and allow you to pick your default region ([![](GKE%20Kubernetes%20Cluster%20-%20Stephen%20Perciballi%20-%20Confluence/favicon.1.ico)Global Locations - Regions & Zones  |  Google Cloud](https://cloud.google.com/about/locations/) ).

## Step 2

Create a custom network. The name should include your Okta username

`gcloud compute networks create sperciballi-network \`  
`--subnet-mode custom`

## Step 3

Create the subnet and specify where you which region you want the network to exist. The subnet can be any RFC1918, but 192.168 is the correct one to use for lab.

`gcloud compute networks subnets create sperciballi-subnet \`  
`--network sperciballi-network \`  
`--region northamerica-northeast1 \`  
`--range 192.168.1.0/24`

## Step 4

Create the cluster. Ensure to modify the variables that have been created in previous steps.

The --master-authorized-networks is your public IP address from the network you’re working from. This is basically a firewall to only permit traffic to the cluster from that IP address.

`gcloud container clusters create sperciballi-cluster \`  
`--num-nodes=2 \`  
`--zone northamerica-northeast1-a \`  
`--create-subnetwork name=sperciballi-subnet \`  
`--enable-ip-alias \`  
`--enable-master-authorized-networks \`  
`--enable-private-nodes \`  
`--master-ipv4-cidr 172.16.0.0/28 \`  
`--master-authorized-networks 69.165.153.0/24`  
`--network "projects/snyk-cx-se-demo/global/networks/sperciballi-network" \`  
`--subnetwork "projects/snyk-cx-se-demo/regions/northamerica-northeast1/subnetworks/sperciballi-subnet"`

## Step 5 - Credentials

Pull the credentials for the cluster in GKE to your local kubectl.

`gcloud container clusters get-credentials sperciballi-cluster --region northamerica-northeast1-a`

## Step 6 - Connect to the Internet

Create a router for your network:

`gcloud compute routers create sperciballi-router \`  
`--network sperciballi-network \`  
`--region northamerica-northeast1`

Setup network address translation so that all traffic will egress a single public IP address:

`gcloud compute routers nats create sperciballi-nat-config \`  
`--router-region northamerica-northeast1 \`  
`--router sperciballi-router \`  
`--nat-all-subnet-ip-ranges \`  
`--auto-allocate-nat-external-ips`
