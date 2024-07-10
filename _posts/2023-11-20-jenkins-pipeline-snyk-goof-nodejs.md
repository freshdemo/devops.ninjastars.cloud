---
layout: post
title:  "Jenkins Pipeline - Snyk Goof Node.js"
date:   2023-11-20 21:27:07 -0400
tags: infrastructure jenkins cicd pipeline
---

This pipeline will build and scan Snyk Goof Node.js.

## Step 1 - Deploy Jenkins

If you haven’t already deployed Jenkins here is a guide, [Jenkins Container](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/20/1757020255).

## Step 2 - Install Node.js Plugin

Navigate to Dashboard, Manage Jenkins, Plugins. From Available plugins search for Node.js and install.

## Step 3 - Specify the Node.js Version

In this case we want Node.js version 16 as documented in the Snyk Goof Node.js build guide, [Code Repository - Snyk Goof Node.js](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/17/1754398813).

Navigate to Manage Jenkins, Tools, NodeJS, and click Add NodeJS.

Give it a name like NodeJS16 and choose the latest version within 16, which should be 16.20.2, and click save.

## Step 4 - Add a SNYK\_TOKEN Environment Variable

This will keep secrets out of your pipeline configuration and keep them as environment variables in Jenkins instead.

Navigate to Manage Jenkins, Credentials, System, Global credentials, and click Add Credentials.

Choose Secret text as the type, enter your Snyk API token available from the Snyk UI under your account settings, and give it an ID of SNYK\_TOKEN.

## Step 5 - Create a Pipeline

From the Dashboard, select new, specify a name (node-goof), and choose Pipeline.

Some modifications you may want to make from the following script that have been included to demonstrate some of the most important options available:

-   Line 20 - While not necessary, change this to your own repo.
    
-   Line 37 - Note you don’t have to authenticate snyk manually when the variable SNYK\_TOKEN is specified but wanted to highlight how.
    
-   Line 58 - Only medium severity and greater vulnerabilities are being included for Open Source.
    
-   Line 65 - An HTML report artifact will be attached to the pipeline for Snyk Code results once complete, and only medium severity and greater vulnerabilities are being included.
    
-   Line 73 - While not necessary, change this to your own container name.
    
-   Line 73 - The Snyk Container test will not break the build in this pipeline due to the || true on the end.
    

`// Jenkinsfile solution based on the pipeline plugin: // https://www.jenkins.io/solutions/pipeline/ // Results are output in SARIF format & processed using the Warnings NG plugin: // https://plugins.jenkins.io/warnings-ng/ // Please read the contents of this file carefully & ensure the URLs, tokens etc match your organisations needs. pipeline { agent any environment { SNYK_TOKEN = credentials('SNYK_TOKEN') } // Requires a configured NodeJS installation via https://plugins.jenkins.io/nodejs/ tools { nodejs "NodeJS16" } stages { stage('git clone') { steps { git url: 'https://dev.azure.com/stephenperciballi/Development/_git/node-goof' } } // Install the Snyk CLI with npm. For more information, check: // https://docs.snyk.io/snyk-cli/install-the-snyk-cli //stage('Install snyk CLI') { // steps { // script { // sh 'npm install -g snyk' // } // } //} // Authorize the Snyk CLI //stage('Authorize Snyk CLI') { // steps { // script { // withCredentials([string(credentialsId: 'SNYK_CREDS', variable: 'SNYK_CREDS')]) { // sh 'snyk auth ${SNYK_TOKEN}' // } // } //} stage('Build App') { steps { // Replace this with your build instructions, as necessary. sh 'npm install' sh 'npm install snyk-to-html -g' } } stage('Snyk') { parallel { stage('Snyk Open Source') { steps { catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') { sh 'snyk test --severity-threshold=medium' } } } stage('Snyk Code') { steps { catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') { sh 'snyk code test --json --severity-threshold=medium | snyk-to-html -o ./results-code.html' sh 'snyk code test --severity-threshold=medium' } } } stage('Snyk Container') { steps { catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') { sh 'snyk container test freshdemo/todolist-goof --file=Dockerfile || true' } } } // stage('Snyk IaC') { // steps { // catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') { // sh 'snyk iac test' // } // } // } } post { always { archiveArtifacts artifacts: '*.html' } } } } }`

## Step 6 - Run the Pipeline

Click Build Now to run the pipeline. From the Pipeline, Status you will see each stage represented by a pass/fail.

Note the Snyk Code HTML report also will be displayed under Pipeline, Status.

## Troubleshooting

From Pipeline, Status you can mouse over each numbered build stage and get a popup where you can navigate directly to the logs for each stage.

From the same area you can select the down pointing arrow next to the pipeline run number and select Console Output to show all of the logs.

Be the first to add a reaction