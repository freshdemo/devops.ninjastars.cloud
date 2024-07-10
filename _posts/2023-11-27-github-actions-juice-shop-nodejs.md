---
layout: post
title:  "GitHub Actions Pipeline - Juice Shop Node.js"
date:   2023-11-27 21:27:07 -0400
tags: infrastructure github-actions cicd pipeline
---
This pipeline will build and scan Juice Shop.

You can modify any of these steps if you choose to use another repository.

-   get the code
    
-   set the correct package manager
    
-   build the app
    
-   scan with Snyk
    

## Step 1 - Create a GitHub Account

You should have this already,[![](GitHub%20Actions%20-%20Juice%20Shop%20Node.js%20-%20Stephen%20Perciballi%20-%20Confluence/fluidicon.png)GitHub: Let’s build from here](https://www.github.com/).

## Step 2 - Create a Repo

For this use case we’ll use Juice Shop, [Code Repository - Juice Shop](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1749254232) . There is important information about versions and the build in this document so take a look.

-   From the `Overview` page, click on Repositories, and the green `Add` button on the right.
    
-   Set the name and configure privacy settings to suit, and leave the rest of the defaults as we will have these in the repo already. Click `Create repository`.
    
-   Assuming you followed the steps here, [Code Repository - Juice Shop](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1749254232), cd into your local Juice Shop directory. Issue the following commands:
    
    -   `git remote remove origin`
        
    -   `git remote add origin <the origin provided on the GitHub page>`
        
    -   `git branch -M main`
        
    -   `git push -u origin main`
        

## Step 3 - Create a Secret

Set a Repository Secret for your Snyk API token.

1.  Navigate to the repo in GitHub.
    
2.  Select `Settings`.
    
3.  Select `Secrets and Variables`, then select `Actions`.
    
4.  Create a `New repository secret.`
    
5.  Call it `SNYK_TOKEN`, and use the API token from the Snyk UI under account settings.
    

## Step 4 - Create a Pipeline

While building/tuning a pipeline it can be easier to edit from the GitHub UI, and switch between the yml code and the Actions tab so that you can see the run without having to switch windows. Some people prefer to edit the file locally in their IDE or terminal, then commit the change to GitHub, then go see the results in the Actions tab.

1.  Navigate to the `Code` tab, and drill into the `.github`, `workflows`, directories. Alternatively you can edit the files locally on your laptop and push the changes.
    
2.  Rename or remove the .yml files. There are some impressive pipelines here that are worth learning from, however they are going to slow down your builds considerably.
    
3.  Edit the .gitignore and comment out package-lock.json.
    
4.  Use something similar to the following yaml. You can create a file in .github/workflows called snyk-security.yml. Here are some details about the file you may choose to modify;
    
    1.  Line 5 is monitoring the main branch for changes. If you are using a different branch name, set that here.
        
    2.  Line 11, the permissions are to push Snyk Code can results to the GitHub Security tab. Some organizations will want to see this.
        
    3.  Line 19, the version of Node.js from the post on Juice Shop [Code Repository - Juice Shop](https://snyksec.atlassian.net/wiki/spaces/~629db3cb76c0360069f263e7/blog/2023/11/16/1749254232) is being set. Failing to set this may cause the build to fail.
        
    4.  Line 29, build the app with npm.
        
    5.  All of the test results will be available in the GitHub Actions steps. If you prefer to have them in the Snyk UI modify them to use the various `monitor` options from the Snyk CLI.
        
    6.  All of the tests except container are currently piped to true, which causes them to not fail. You may want to fail based on --severity-threshold or just fail on any alert.
        

`name: Snyk Security CI on: push: branches: [ main ] pull_request: branches: [ main ] jobs: build: permissions: contents: read security-events: write actions: read runs-on: ubuntu-latest strategy: matrix: node-version: [18.x] # See supported Node.js release schedule at https://nodejs.org/en/about/releases/ steps: - uses: actions/checkout@v3 - name: Use Node.js ${{ matrix.node-version }} uses: actions/setup-node@v3 with: node-version: ${{ matrix.node-version }} cache: 'npm' - run: npm install - run: npm i -g snyk - run: snyk auth ${{ secrets.SNYK_TOKEN }} - name: install snyk-to-html run: | npm install snyk-to-html -g - name: Snyk Open Source # For testing and failing please add snyk test before snyk monitor run: | snyk test --json | snyk-to-html -o results-opensource.html || true - name: Use the Upload Artifact GitHub Action uses: actions/upload-artifact@v3 with: name: results-opensource path: results-opensource.html - name: Snyk Code # Remove || true to fail if there are vulnerabilities run: | snyk code test --json | snyk-to-html -o results-code.html || true snyk code test --sarif > snyk-code.sarif || true # - name: Use the Upload Artifact GitHub Action # uses: actions/upload-artifact@v3 # with: # name: results-code # path: results-code.html - name: Upload result to GitHub Code Scanning uses: github/codeql-action/upload-sarif@v2 with: sarif_file: snyk-code.sarif - name: Snyk IaC # Remove || true to fail if there are vulnerabilities run: | snyk iac test --report || true - name: Snyk Container # Rename your image, for testing and failing please add snyk container test before snyk container monitor run: | snyk container test freshdemo/juice-shop-github-dockerhub-gke:actions --file=Dockerfile`

### Troubleshooting

1.  Navigate to the repo in GitHub.
    
2.  Select Actions from the top menu, where you will find a list of the Actions runs named after commits.
    
3.  Click on the folder under the filename (i.e. snyk-security.yml), then on the next folder.
    
4.  From this view you will see each of the steps. You can click on each of them to see exactly what happened, and this is where you’ll find the most valuable error messages.
    

Be the first to add a reaction