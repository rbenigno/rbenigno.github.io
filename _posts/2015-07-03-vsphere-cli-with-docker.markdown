---
layout: post
title:  "vSphere CLI with Docker (almost)"
date:   2015-07-03 22:16:40 -0800
categories: vsphere docker
---
# Overview

The lack of native OS X tools for vSphere has always been a source of frustration for me.  When building vSphere environments I usually turned to PowerCLI over RDP.  This works okay, but jumping into a Windows machine definitely slows me down.

While not OS X native, the [Docker Container for VMware Utilities](http://www.virtuallyghetto.com/2015/02/cool-docker-container-for-vmware-utilities.html) put together by 
[William Lam](http://www.virtuallyghetto.com/about) and [Alan Renouf](http://www.virtu-al.net) is close enough.

## Setup
Head over to the [blog post](http://www.virtuallyghetto.com/2015/02/cool-docker-container-for-vmware-utilities.html) or [Github repo](https://github.com/alanrenouf/vmware-utils) for more details on the container.  In this post I will cover both the Docker setup and vmware-utils container.

You will need VMware Fusion or VirtualBox for the Docker host.

**Step 1** - You should already have this; if not, open a [terminal](https://www.iterm2.com) and install [Homebrew](http://brew.sh).

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    
**Step 2** - Install [Docker](https://www.docker.com/whatisdocker) and [Docker Machine](https://docs.docker.com/machine/).

    brew install docker docker-machine

**Step 3** - Create and start your Docker host.

    docker-machine create --driver vmwarefusion tools
   
   *Or*
    
    docker-machine create --driver virtualbox tools

**Step 4** - Setup your terminal session so the Docker client targets your new machine.

    eval "$(docker-machine env tools)"

**Step 5** - Grab the Dockerfile from Github, and create the working folder.

    git clone https://github.com/alanrenouf/vmware-utils.git
    cd vmware-utils

**Step 6** - Download the [requisite tools](https://github.com/alanrenouf/vmware-utils#how) from VMware and place in the vmware-utils folder with the Dockerfile.

**Step 7** - Build the container.

    docker build -t vmware-utils .

**Step 8** - Run the container.

    docker run --rm -it vmware-utils

## Usage

After the setup is done you will be able to quickly launch your machine and access the container.

    docker-machine start tools
    eval "$(docker-machine env tools)"
    docker run --rm -it vmware-utils

When done...

    docker-machine stop tools 

Here is an example of running with environmental variables set for esxcli:

    docker run --rm -it  \
     -e VI_SERVER=<vcenter.domain.tld> \
     -e VI_USERNAME=<myuser> \
     vmware-utils

You can also run a command and exit:

    docker run --rm -it vmware-utils esxcli -s <vcenter.domain.tld> -u <myuser> -h <esxhost> vm process list