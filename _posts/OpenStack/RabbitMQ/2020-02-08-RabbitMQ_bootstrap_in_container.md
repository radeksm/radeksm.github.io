---
layout: post
title: RabbitMQ bootstrap in container
tags:
    - RabbitMQ
    - podman
author: Radosław Śmigielski
---

Why?
====
Podman or Docker is yet another way of running RabbitMQ and in some cases it can have some adventages.
I use Podman in below example but if you prefer docker just replace **podman** by **docker** commands.
I personally prefer Podman because it does not requre root proviliges to run docker containers.

How?
====

The simplest possible configuraton
----------------------------------
Download RabbitMQ container and run it.
```bash
podman pull docker.io/library/rabbitmq:latest

RABBITMQ_ERLANG_COOKIE="SomeRandomStringHere"
HOSTNAME="rabbit00"

podman run -d \
    --hostname ${HOSTNAME}.kresy.local \
    --name ${HOSTNAME} \
    -p 5672:5672 \
    -e RABBITMQ_ERLANG_COOKIE=${RABBITMQ_ERLANG_COOKIE} \
    -e RABBITMQ_NODENAME=rabbit@${HOSTNAME} \
    docker.io/library/rabbitmq:latest
```
Enter your new RabbitMQ container.
```bash
podman  exec -it rabbit00 bash
```

RabbitMQ with management plugin enabled
---------------------------------------
Download RabbitMQ container with management plugin and run it.
```bash
podman pull docker.io/library/rabbitmq:management

RABBITMQ_ERLANG_COOKIE="SomeRandomStringHere"
HOSTNAME="rabbit01"

podman run -d \
    --hostname ${HOSTNAME}.kresy.local \
    --name ${HOSTNAME} \
    -p 5672:5672 \
    -p 15672:15672 \
    -e RABBITMQ_ERLANG_COOKIE=${RABBITMQ_ERLANG_COOKIE} \
    -e RABBITMQ_NODENAME=rabbit@${HOSTNAME} \
    docker.io/library/rabbitmq:management

```

Set up admin user and password for management plugin.
```bash
podman  exec -it rabbit01 bash

# below commands need to be executed inside container
rabbitmqctl add_user admin password
rabbitmqctl set_user_tags admin administrator
```
Open RabbitMQ management plugin in your browser at
[http://localhost:15672/](http://localhost:15672/)
