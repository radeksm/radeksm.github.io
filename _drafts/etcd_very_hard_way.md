---
layout: post
title: Settig up etcd cluster - very hard way
tags:
    - etcd
author: Radosław Śmigielski
---

Why?
====
I wanted to start playing with etcd on my personal laptop, there is a number
of existing guides how to setup an etcd cluster. But none of these was what
I was looking for. Even famous Kelsey Hightower
[kelseyhightower/kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
was not what I was looking for.

My way let me:
1. run etcd instances on my laptop
2. does not require docker or any other infrastructure, external VMs
3. internal to my laptop only

Network
=======
Below bash script creates internal network of 3 dummy interfaces to which
my etcd instances will be binded. I also add these IP addresses to hosts
file.
{% highlight bash %}
# as a root
ip link add dummy1 type dummy
ip link add dummy2 type dummy
ip link add dummy3 type dummy

ip a a 10.255.255.1/25 dev dummy1
ip a a 10.255.255.2/25 dev dummy2
ip a a 10.255.255.3/25 dev dummy3

cat << EOF >> /etc/hosts
######## etcd cluster ########
10.255.255.1   etcd1
10.255.255.2   etcd2
10.255.255.3   etcd3
EOF
{% endhighlight %}

Bootstrap cluster
=================

First cluster member
--------------------
{% highlight bash %}
#!/bin/bash

BASE='/home/radek/src/my_staff/etcd/etcd-v3.3.5-linux-amd64'
ETCD="${BASE}/etcd"

token=$(pwgen -s 20)
token=ohMo0eichu9auxo5yath

name="etcd1"
peer_port=2380
clent_port=2379
inst_data="/tmp/etcd/$name"

[ -d "$inst_data" ] && rm -rf $inst_data
mkdir -p $inst_data/data

${ETCD} \
    --name $name \
    --data-dir $inst_data/data \
    --listen-peer-urls http://10.255.255.1:$peer_port \
    --listen-client-urls http://10.255.255.1:$clent_port \
    --advertise-client-urls http://$name:$clent_port\
    --initial-cluster-state new \
    --initial-advertise-peer-urls http://$name:$peer_port \
    --initial-cluster etcd1=http://etcd1:$peer_port,etcd2=http://etcd2:$peer_port,etcd3=http://etcd3:$peer_port \
    --initial-cluster-token $token

{% endhighlight %}

Second cluster member
--------------------
{% highlight bash %}
{% endhighlight %}

Third cluster member
--------------------
{% highlight bash %}
{% endhighlight %}

