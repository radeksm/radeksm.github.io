---
layout: post
title: Chage docker daemon group
tags:
    - docker
    - fedora
    - systemd
author: Radosław Śmigielski
---

Why?
====
On some linux distros Docker daemon runs as __*root:root*__ which
is not always what you want on your dev machine.
1. you need to use root account to work with docker
1. let's do this in the right way not a hacky way using systemd,
   so no upgrade will break it
1. if you compile projects like Kuberenetes you need running Docker
   and you really do not want to compile staff from under root account

Why not?
========
On real production system you probably want to leave docker running
under root acount because docker group grants privileges almost equivalent
to the root user. More on that topic
[Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)


How?
====

Below you can find how to run docker under different than root group
and let you add normal users to __*docker*__ group and not to use root
account to work with docker.

1. Create the docker group and add your user to that new group
   ```
   sudo groupadd docker
   sudo usermod -aG docker $USER
   ```
1. Logoff and login $USER in order to $USER becomes docker group member.
1. Take distro docker service systemd file usually from
   __*/usr/lib/systemd/system/docker.service*__ and copy it to
   __*/etc/systemd/system/docker.service*__.
   This file will overwrite existsing docker systemd config file and let
   you customize it and make sure it will be persistent over next
   docker daemon upgrade.
   ```
   cp /usr/lib/systemd/system/docker.service /etc/systemd/system/docker.service
   ```
1. Use __*--group*__ option which:
   ```
   -G, --group=""
       Group to assign the unix socket specified by -H when running in daemon mode.
       use '' (the empty string) to disable setting of a group. Default is docker.
   ```
   Edit your custom docker service file __*/etc/systemd/system/docker.service*__
   and in section __*ExecStart*__ add __*--group*__ option
   ```
   ExecStart=/usr/bin/dockerd-current \
          ... 
          --group docker \
          ... 
   ```
1. Reload systemd config files
   ```
   systemctl daemon-reload
   ```
1. Verify your new docker service file is picked up by docker daemon
   ```
   systemd-delta --type=overridden
   ```
1. Restart docker.service
   ```
   systemctl restart docker.service
   ```
1. Validate your $USER can suncesfully execute docker commands
   and docker unix socket had a desired group.
   ```
   $ ls -al /var/run/docker.sock
   srw-rw----. 1 root docker 0 Sep 10 04:59 /var/run/docker.sock
   ```




