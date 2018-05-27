---
title: "Deploying a Hugo site to Azure App Services with VSTS and Github"
date: 2018-05-27T16:31:27+03:00

categories: ['Code', 'Tutorials']
tags: ['Hugo', 'Azure', 'App Services', 'VSTS']
author: "Petre"
---
After some research, I've decided to create this blog using the awesome [Hugo](https://gohugo.io) and the wonderful [Bilberry Theme](https://github.com/Lednerb/bilberry-hugo-theme).
For ease of writing, I have setup an automated build process using VSTS which includes deploying to an Azure App Service. This is a step-by-step guide of what I've done.

<!--more-->

For the purpose of this guide, I'll assume you have Hugo installed locally and already created a site.
If you need a guide on how to start using Hugo, please follow to Getting Started guide here: https://gohugo.io/getting-started/

__Setting up VSTS:__

- Create a new project in your VSTS team:
    https://yourvststeam.visualstudio.com/_projects?_a=new

For this guide, we'll use Github as the code repository and solely use VSTS for building and deploying the code.

- Set up a build definition:
    Navigate to Build and Release > Builds and select "New Definition"
    