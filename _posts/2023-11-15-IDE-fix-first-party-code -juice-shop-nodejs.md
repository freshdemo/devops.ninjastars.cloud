---
layout: post
title:  "Fix First Party Code - Juice Shop"
date:   2023-11-15 21:27:07 -0400
tags: fix code node.js juice-shop
---
1.  With your project open in the IDE, navigate to the Snyk plugin on the bottom left, and to the Code Security section. Notice there are 14 issues in vulnCodeSnippets.ts. There should also be ~263 total Snyk Code vulnerabilities.
    
2.  Select the first issue, “Unsanitized user input from an HTTP parameter”. 
    
3.  On the right you should see the Issue card for Snyk Code Vulnerability open. Notice that under the name you are provided with the line number the vulnerability is present. Also notice the example fixes in this pane.
    
4.  On line 104 also notice the issue is underlined in red. Mouse over the issue to view the full description of the vulnerability.
    
5.  Navigate to line 104 of our vulnerable vulnCodeSnippets.ts file. Currently the line looks like this; 
    

<img src="/images/fix-juice1.png">

<img src="/images/fix-juice2.png">


``const match = new RegExp(`vuln-code-snippet start.*${challenge.key}`)``

6.  Modify the line to use sanitizeHtml by changing it to look like 
    

``const match = new RegExp(`^${_.escapeRegExp(vuln-code-snippet start.*${challenge.key})}`)``

7.  Navigate to File, Save.
    
8.  Save the file by selecting File, Save. Snyk should re-scan the code automatically. If not, hit the play button in the Code Security section.
    
9.  Notice the vulnerability for Unsanitized User HTTP input is now gone, and the total count has gone from ~263 to 255 because this resolved several other issues.
    

<img src="/images/fix-juice3.png">
