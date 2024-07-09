---
layout: post
title:  "Gitlab Broker"
date:   2023-05-19 21:27:07 -0400
categories: [gitlab, infrastructure, scm, broker]
---
The best place to start is with the official docs as things may change, [![](GitLab%20Broker%20-%20Stephen%20Perciballi%20-%20Confluence/spaces%252F-MdwVZ6HOZriajCf5nXH%252Favatar-1631192016346.png)GitLab - install and configure using Docker | Snyk User Docs](https://docs.snyk.io/enterprise-setup/snyk-broker/install-and-configure-snyk-broker/gitlab-install-and-configure-broker/setup-broker-with-gitlab) .

## Step 1 - Create a Broker Token

Get your _Snyk API_ token by navigating to the Snyk UI, click on your name, and select Account Settings.

Get the _Organization ID_ by navigating to the Snyk UI, click on your Organization, and select Settings.

Run the following command from your laptop:

`curl --include \`  
`--header "Content-Type: application/json" \`  
`--header "Authorization: token <SNYK_TOKEN>" \`  
`'https://api.snyk.io/api/v1/org/<organization ID>/integrations/gitlab'`

## Step 2 - Create a GitLab PAT

Create a GitLab Personal Access Token by following these steps in your environment, [![](GitLab%20Broker%20-%20Stephen%20Perciballi%20-%20Confluence/favicon.ico)Personal access tokens | GitLab](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) .

## Step 3 - Deploy the Broker Container

Here is where the variables below come from:

-   `BROKER_TOKEN` comes from step 1
    
-   `GITLAB_TOKEN` is from step 2
    
-   `GITLAB` is from the hostname specified while deploying GitLab [Gitlab container](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/10/18/1715732548) .
    
-   `BROKER_CLIENT_URL` is the internal IP address of your laptop.
    

`docker run --restart=always \`  
`-p 8000:8000 \`  
`-e NODE_TLS_REJECT_UNAUTHORIZED=0 \`  
`-e BROKER_TOKEN=<broker token from admin panel> \`  
`-e GITLAB_TOKEN=<GitLab PAT from GitLab> \`  
`-e GITLAB=gitlab.perciballi.ca \`  
`-e PORT=8000 \`  
`-e BROKER_CLIENT_URL=http://192.168.1.76:8000 \`  
`-e ACCEPT_CODE=true \`  
`snyk/broker:gitlab`

## Step 4 - Verify Connectivity

The easiest way to validate Broker is working is to navigate to the Snyk UI, Project, Add Projects, and select GitLab. You should see a list of repositories stored in your local GitLab.

## Troubleshooting

Typical Broker troubleshooting applies. Firewalls and SSL Decryption are the most common culprits so if you have those security systems in your environment permit the traffic there.

Curl can provide some valuable information

`curl -isk https://broker.snyk.io`

You can also look at the Broker container logs. To find the _Container ID_ run;

`docker ps -a`

Then check the logs with

`docker logs -f <your container ID>`

Be the first to add a reaction