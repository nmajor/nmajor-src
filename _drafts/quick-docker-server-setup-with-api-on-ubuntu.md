---
layout: post
title: Quick Docker Server Setup with API on Ubuntu
hero: ''
tags:
- docker
date: ''

---
This is mostly just a quick reference for me to use when I need to whip up a docker server. This is on Ubuntu 16.04.

# Install Docker

First prepare the app registry:

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt-get update

Then check the output of this command. It should have no docker installed but there should be a candidate:

    apt-cache policy docker-ce
    
    # docker-ce:
    #   Installed: (none)
    #   Candidate: 5:18.09.0~3-0~ubuntu-xenial
    #   Version table:
    #      5:18.09.0~3-0~ubuntu-xenial 500
    #   ...

Then install docker:

    sudo apt-get install -y docker-ce

Then check the output of this command. Docker should be loaded and active:

    sudo systemctl status docker
    
    # ● docker.service - Docker Application Container Engine
    #    Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
    #    Active: active (running) since Fri 2019-01-04 22:45:48 UTC; 1min 22s ago
    #      Docs: https://docs.docker.com
    #  Main PID: 3538 (dockerd)
    #    CGroup: /system.slice/docker.service
    #            └─3538 /usr/bin/dockerd -H unix://
    # 
    # .... Logs down here should say something like Started Docker Application Container Engine. at some point

# Configure Docker API

Here is where we tell docker to listen to incoming API requests.

Make a new `/etc/systemd/system/docker.service.d/docker.conf` file and open it for editing:

    sudo mkdir /etc/systemd/system/docker.service.d
    sudo touch /etc/systemd/system/docker.service.d/docker.conf
    sudo vim /etc/systemd/system/docker.service.d/docker.conf

Add this to the file:

    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

Restart docker service:

    sudo systemctl daemon-reload
    sudo systemctl restart docker.service

Check output of this command. It should have `-H tcp://0.0.0.0:2375` in the docker command.

    sudo systemctl status docker.service
    
    # ● docker.service - Docker Application Container Engine
    #    Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
    #   Drop-In: /etc/systemd/system/docker.service.d
    #            └─docker.conf
    #    Active: active (running) since Fri 2019-01-04 22:54:49 UTC; 3s ago
    #      Docs: https://docs.docker.com
    #  Main PID: 3791 (dockerd)
    #     Tasks: 8
    #    Memory: 30.0M
    #       CPU: 178ms
    #    CGroup: /system.slice/docker.service
    #            └─3791 /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
    #
    # .... Logs down here should say something like Started Docker Application Container Engine. at some point          