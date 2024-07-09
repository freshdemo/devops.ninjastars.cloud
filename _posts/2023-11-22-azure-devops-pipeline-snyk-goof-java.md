---
layout: post
title:  "Azure Devops Pipeline - Snyk Goof Java"
date:   2023-11-22 21:27:07 -0400
categories: infrastructure azuredevops cicd pipeline
---

This pipeline will build and scan Snyk Goof Java.

You can modify any of these steps if you choose to use another repository.

-   get the code
    
-   set the correct package manager
    
-   build the app
    
-   scan with Snyk
    

## Step 1 - Create a Free Azure Account

You can use your [outlook.com](http://outlook.com/ "http://outlook.com") if you already have one, or they now support GitHub logins, [Azure DevOps Services | Microsoft Azure](https://azure.microsoft.com/en-us/products/devops/?nav=min) .

You’ll immediately need to create a Project, which is similar to a Snyk Organization. You’ll be able to have multiple repositories and pipelines in the Project.

## Step 2 - Create a Repo

For this use case we’ll use Snyk Goof Java, [Code Repository - Snyk Goof Java](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/17/1754431492). There is important information about versions and the build in this document so take a look.

-   From your Project, click on Repos, and Add new repo. If you already have a repo the workflow to add a new one will be in the top menu where you’ll see your Organization, Project, Repos, Files, and a repo name. Click the drop down arrow there to add a new Repo. Don’t add a README.
    
-   In this case you will follow the steps to **Push an existing repository from command line**. Because you will have cloned java-goof (or whatever repo name) from somewhere else cd into the directory of the repo you want to push to Azure and run `git remote delete origin` prior to following the steps. This will remove the original repo destination and allow you to set a new one to Azure.
    

## Step 3 - Install a Build Agent

All of the CI tools today use a build agent/runner. The build agent/runner is a system for your pipeline script commands to run. At the time of this writing, Microsoft does not provide a cloud based build agent/runner on free accounts. The solution to this is to download the build agent and install it on your laptop. The instructions from Microsoft is probably the best place to start, [Deploy a build and release agent on macOS - Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/osx-agent?view=azure-devops), but following are the steps:

1.  Navigate to the Azure DevOps Organization, which is the top level URL like [https://dev.azure.com/](https://dev.azure.com/ "https://dev.azure.com/"){your organization ID}, then to Organization Settings at the bottom, and Agent Pools.
    
2.  Click on the Default agent, and select Agents.
    
3.  Click New Agent in the top right. This will walk through the process of downloading and installing the Agent.
    
4.  Personally I prefer to run the agent interactively with `run.sh` rather than installing the service, because your laptop is not likely going to need to be running pipelines all day.
    

## Step 4 - Create a Pipeline

**All of the steps will run locally on your system so you’ll need Java** [Laptop Install Java & maven](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/15/1750204429)**, and the Snyk CLI installed.**

1.  Navigate to Piplines from the left menu, and click New Pipeline on the upper right.
    
2.  Select Azure Repos as the source, and choose your repo.
    
3.  Select Starter Pipeline.
    
4.  Select Variables in the top right. Create the following variables;
    
    1.  BuildArtifactStagingDirectory = the directory on your system the local copy of the repository can be found.
        
    2.  BuildStagingDirectory = the directory on your system the local copy of the repository can be found.
        
5.  Use something like the code block below in the yaml file box. Of note:
    
    1.  \# are comments.
        
    2.  Trigger is on the main branch. You can change this or even create an array of branches.
        
    3.  The pool name Default is from step 3.
        

`# Starter pipeline # Start with a minimal pipeline that you can customize to build and deploy your code. # Add steps that build, run tests, deploy, and more: # https://aka.ms/yaml trigger: - main pool: name: Default steps: - script: | export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home" mvn install displayName: Build App - script: | cd /Users/stephen/Documents/java-goof snyk monitor --org=todolist --all-projects displayName: 'Snyk Open Source' - script: | cd /Users/stephen/Documents/java-goof snyk code test --org=todolist --json | snyk-to-html -o /Users/stephen/Documents/java-goof/results-code.html displayName: 'Snyk Code' - script: | cd /Users/stephen/Documents/java-goof snyk iac test --org=todolist --report || true displayName: 'Snyk IaC' - script: | cd /Users/stephen/Documents/java-goof snyk container monitor freshdemo/todolist-goof:latest --file=/Users/stephen/Documents/java-goof/todolist-goof/Dockerfile || true displayName: 'Snyk Container' - script: | cd /Users/stephen/Documents/java-goof/todolist-goof docker buildx build --platform linux/amd64 --file ./Dockerfile -t northamerica-northeast2-docker.pkg.dev/snyk-cx-se-demo/sperciballi-registry/java-goof:gke --push . displayName: 'Build Image' - script: docker push northamerica-northeast2-docker.pkg.dev/snyk-cx-se-demo/sperciballi-registry/java-goof displayName: 'Push Image' - task: CopyFiles@2 inputs: SourceFolder: '$(BuildStagingDirectory)' Contents: 'results-code.html' TargetFolder: '$(BuildStagingDirectory)' - publish: $(BuildStagingDirectory)/results-code.html artifact: results-code.html`

### Troubleshooting

Navigate to Pipelines, and Runs. Choose the run, then at the bottom click job. From here you can see each step, and selecting a step will give you details about the run. You’ll be able to see things like Snyk scan data and errors in each step.

For the most part you’ll be looking for errors about the wrong build tool (or lack thereof), lack of authorization (say for the Snyk scans), and scripting errors in the pipeline (like missing brackets).

Be the first to add a reaction