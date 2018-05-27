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

### 1. Prerequisites

For the purpose of this guide, I'll assume you have Hugo installed locally and already created a site.
If you need a guide on how to start using Hugo, please follow to Getting Started guide here: https://gohugo.io/getting-started/

You will also need a new VSTS project in your team.
For this guide, we'll use Github as the code repository and solely use VSTS for building and deploying the code.

### 2. Setting up the build definition

##### 2.1. Selecting the source

Navigate to Build and Release > Builds and select `New Definition`
Select `Github` as a source and then `Authorize VSTS`
Select your repository and branch:

![Repo and branch](/images/2018-05-27-17-17-13.png)

Hit `Continue` and select `Empty process` on the next screen as we won't be using a template

##### 2.2. Managing themes

If you are following any Hugo theme installing guide online, one of the required step is to `git clone` the theme into the `/themes/` subfolder. When pushing your project to a Git repository, such as Github, that cloned theme will be considered a submodule.

To enable VSTS to resolve the submodule as well:

- Add a `.gitmodules` file to your project containing:

```git
[submodule "theme"]
    path = themes/bilberry-hugo-theme
    url = https://github.com/Lednerb/bilberry-hugo-theme.git
```

- Commit your changes to Github
- Select the `Get sources` step in VSTS and check `Checkout submodules`

![Checkout submodules](/images/2018-05-27-19-16-36.png)

##### 2.3. Setting up the build agent

We are using a Linux agent for this guide, so go ahead and select the Process step on the left and select the `Linux agent` queue in the `Agent Queue` dropdown list.

![Build Agent Setup](/images/2018-05-27-17-33-25.png)

For the build agent, we need to install Hugo. According to the [Official documentation](https://gohugo.io/getting-started/installing/), you have a choice of either using Snap or apt-get. Snap will not work in our scenario and apt-get will probably resolve to and older version. Instead, we will use the [Release](https://github.com/gohugoio/hugo/releases) page of the official repository and use the latest debian package to install Hugo.

Head over to https://github.com/gohugoio/hugo/releases and copy the link of the latest x64 Linux deb package. When this article was written, the latest version was 0.41 so we'll use:
`https://github.com/gohugoio/hugo/releases/download/v0.41/hugo_0.41_Linux-64bit.deb`

Create a new Task in VSTS of type `Command Line`
For `Tool` type `wget`, and for `Arguments` type `-O hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.41/hugo_0.41_Linux-64bit.deb`

![wget](/images/2018-05-27-19-23-09.png)

Create another `Command Line` task and type in `dpkg -i hugo.deb`

![dpkg](/images/2018-05-27-19-38-04.png)

This will install Hugo which we'll use to build our blog. 

##### 2.4. Building the site

Add one last `Command Line` task and type in `hugo -d dist -v`
This last command will build our Hugo blog and output the static files to `dist/`. `-v` sets a Verbose output so we'll know if anything goes wrong.

![hugo](/images/2018-05-27-19-38-53.png)

### 3. Set up the deployment

For the current project there is no need for separate build and release steps, so the deployment step will be added to the same build definition.
For this example, we will be using an App Service on Windows as a destination.

##### 3.1. Add the deployment step

Add an Azure App Service Deploy task and follow these steps:

- Select your Azure Subscription
- Authorize VSTS to access the subscription (this will create a Service Principal with deployment access)
- Select the App Service name you wish to deploy to and a slot if required
- For `Package or folder` type in `$(System.DefaultWorkingDirectory)/dist/`

![Deploy](/images/2018-05-27-19-33-58.png)

##### 3.2. Run your deployment

Simply hit `Save & Queue` and watch the magic happen.

### 4. Set up Continuous Integration

To automatically trigger a Build and Deploy process when changes are pushed, open Triggers under the Build definition and check `Enable continuous integration`

![CI](/images/2018-05-27-19-54-46.png)