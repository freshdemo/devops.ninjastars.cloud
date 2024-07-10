---
layout: post
title:  "Fix First Party Code - Goof Node.js"
date:   2023-10-23 21:27:07 -0400
tags: fix code node.js goof
---
For particularly technical prospects they may want to see what it looks like to fix a Snyk Code issue. This use case leverages:

-   Visual Studio Code
    
-   The Snyk Plugin for VS Code.
    
-   The Node.js version of Snyk Goof, https://github.com/snyk-labs/nodejs-goof.
    

1.  With your project open in the IDE, navigate to the Snyk plugin on the bottom left, and to the Code Security section. Notice there are 11 issues in index.js.
    
2.  Select the first issue, “Unsanitized input from an HTTP parameter”. 
    
3.  On the right you should see the Issue car for Snyk Code Vulnerability open. Notice that under the name you are provided with the line number the vulnerability is present. Also notice the 3  example fixes in this pane.
    
<img src="/images/fix-goof-node1.png">
    
4.  On line 116 also notice the issue is underlined in red. Mouse over the issue to view the full description of the vulnerability.
    
5.  We’re going to fix this our own way. First open the VSCode terminal again, and run _npm i sanitize-html_. This installs a package that sanitizes input for us.
    
6.  Navigate to line 116 of our vulnerable index.js file. Currently the line looks like this; `Todo.findById(req.params.id, function (err, todo).` Modify the line to use sanitizeHtml by changing it to look like `Todo.findById(sanitizeHtml(req.params.id, function (err, todo))`. Don’t forget the extra ) at the end.
    
7.  Save the file by selecting File, Save. Snyk should re-scan the code automatically. If not hit the play button in the Code Security section.
    
8.  Notice the vulnerability for Unsanitized HTTP input is now gone, and the count has gone from 11 to 5 because this resolved several other issues.
    
<img src="/images/fix-goof-node2.png">
