---
layout: post
title:  "Snyk Monitor Kubernetes"
date:   2023-10-23 21:27:07 -0400
tags: snyk infrastructure kubernetes
---
This guide walks through deploying Snyk Kubernetes Monitor, providing you’ve already got an on-premise or GKE based ([GKE Kubernetes Cluster](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/10/20/1719238881) ) Kubernetes cluster deployed. Snyk Kubernetes Monitor will provide vulnerability information about the Kubernetes deployment, and the containers running in the cluster, [https://docs.snyk.io/products/snyk-container/kubernetes-workload-and-image-scanning/installation-page/install-the-snyk-controller-with-helm](https://docs.snyk.io/products/snyk-container/kubernetes-workload-and-image-scanning/installation-page/install-the-snyk-controller-with-helm "https://docs.snyk.io/products/snyk-container/kubernetes-workload-and-image-scanning/installation-page/install-the-snyk-controller-with-helm").

## Step 1

Add the helm chart to your system. If you don’t have helm installed you’ll need to install it with `brew install helm`.

`helm repo add snyk-charts https://snyk.github.io/kubernetes-monitor --force-update.`

## Step 2

Create a namespace in Kubernetes dedicated to Snyk Kubernetes Monitor.

`kubectl create namespace snyk-monitor`

## Step 3

Create the secrets required to access the container registry. If the registry is public you don’t need anything in the dockercfg.json. For private registries there are specific configurations for each one that can be found here, [https://docs.snyk.io/scan-applications/snyk-container/integrate-with-kubernetes/use-the-snyk-controller/authenticate-to-private-container-registries](https://docs.snyk.io/scan-applications/snyk-container/integrate-with-kubernetes/use-the-snyk-controller/authenticate-to-private-container-registries "https://docs.snyk.io/scan-applications/snyk-container/integrate-with-kubernetes/use-the-snyk-controller/authenticate-to-private-container-registries").

`kubectl create secret generic snyk-monitor -n snyk-monitor \`  
`--from-literal=dockercfg.json={} \`  
`--from-literal=integrationId=92df0a82-2f54-47b3-9486-5478e3857611 \`  
`--from-literal=serviceAccountApiToken=<Snyk API token>`

## Step 4

Verify the secret was create with;

`kubectl get secrets -n snyk-monitor`

## Step 5

Install the Snyk Kubernetes Monitor on your cluster. Make necessary changes to the namespace if you’ve change the name, and to the cluster name.

`helm upgrade --install snyk-monitor snyk-charts/snyk-monitor \`  
`--namespace snyk-monitor \`  
`--set clusterName="sperciballi-cluster"`

## Step 6

Import the Kubernetes Monitor from the Snyk app by navigating to Projects, Add projects, Kubernetes.

Troubleshooting

If the public IP address in your work environment is dynamic (most home connections) there’s a good chance that it has changed and the ingress firewall in GKI is no longer relevant. Check your current public IP address, then navigate to [https://console.cloud.google.com](https://console.cloud.google.com/ "https://console.cloud.google.com"), Kubernetes Engine, find your cluster, and change `Control plane authorized networks`

<img src="/images/snyk-k8s-monitor.png">

If you are trying to get the secrets working with the container registry typically you’ll want to delete the secret, re-create the secret, then delete the pod.

`kubectl delete secret snyk-monitor -n snyk-monitor`

`kubectl create secret generic snyk-monitor -n snyk-monitor \`  
`--from-literal=dockercfg.json={} \`  
`--from-literal=integrationId=92df0a82-2f54-47b3-9486-5478e3857611 \`  
`--from-literal=serviceAccountApiToken=<Snyk API token>`

`kubectl get all -n snyk-monitor`

`kubectl delete pod snyk-monitor-<pod ID>`

From here the pod will re-create in a minute.
