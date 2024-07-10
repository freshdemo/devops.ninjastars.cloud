---
layout: post
title:  "Laptop - Install Java Maven"
date:   2023-11-15 21:27:07 -0400
tags: infrastructure java maven
---
Getting your developer environment setup is critical for demos. Maven is one of the more popular Java package managers, and requires JDK to run.

## Step 1 - Download & Install JDK

You should be able to use the most recent version of JDK. In this case we’ll use JDK20 from [![](Laptop%20Install%20Java%20&%20maven%20-%20Stephen%20Perciballi%20-%20Confluence/nanoduke.ico)Java Platform, Standard Edition 20 ReferenceImplementations](https://jdk.java.net/java-se-ri/20) .

Unpack the package

`tar -xvf openjdk-11_osx-x64_bin.tar.gz`

Move them into place

`sudo mv jdk-11.jdk /Library/Java/JavaVirtualMachines/`

## Step 2 - Setup System Environment Variables for JDK & Verify

Tell your system where to access JDK by adding the following lines to your ~/.zshrc file. You can also enter these on the terminal to getting it working, but new terminal windows will not have the settings;

`JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-20.0.1.jdk/Contents/Home"`  
`PATH="${JAVA_HOME}/bin:${PATH}"`  
`export PATH`

You can test the installation with;

`java --version`

If you get an error try a new terminal or use `source ~/.zshrc`. Also check the PATH variables set to ensure you’re looking in the right place.

## Step 3 - Download & Install Maven

You should be able to use the most recent version, however check the documentation for the repository you’re working with to verify which version is required [![](Laptop%20Install%20Java%20&%20maven%20-%20Stephen%20Perciballi%20-%20Confluence/favicon.ico)Maven – Download Apache Maven](https://maven.apache.org/download.cgi) .

Unpack the package

`tar -xvf apache-maven-3.6.3-bin.tar.gz`

You can keep this directory anywhere. I left it in ~/Downloads.

## Step 4 - Setup System Environment Variables for Maven & Verify

Tell your system where to access Maven by adding the following lines to your ~/.zshrc file. Put them directly under the JAVA\_HOME variable set above. You can also enter these on the terminal to getting it working, but new terminal windows will not have the settings;

`export M2_HOME="/Users/stephen/Downloads/apache-maven-3.9.2"`  
`PATH="${M2_HOME}/bin:${PATH}"`

You can test the installation with;

`mnv -v`

If you get an error try a new terminal or use `source ~/.zshrc`. Also check the PATH variables set to ensure you’re looking in the right place.
