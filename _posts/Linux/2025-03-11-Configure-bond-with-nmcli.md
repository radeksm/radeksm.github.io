---
layout: post
title: Configure bond interface using Networkmanager nmcli
tags:
    - Linux
    - NetworkManager
author: Radosław Śmigielski
---

Why?
====
The main reason why I created below HOWTO is lack of good documentation
how to create Linux bond interface using Network manager nmcli.

Environment
===========
You need 2 or more NICs, in my case I have 5 physical NICs out of which 3 are only connected.
```bash
root@box ~# ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp9s0f0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 2c:44:fd:8c:e9:ec brd ff:ff:ff:ff:ff:ff
3: enp7s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 04:92:26:da:4b:f2 brd ff:ff:ff:ff:ff:ff
4: enp9s0f1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 2c:44:fd:8c:e9:ed brd ff:ff:ff:ff:ff:ff
5: enp9s0f2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 2c:44:fd:8c:e9:ee brd ff:ff:ff:ff:ff:ff
6: enp9s0f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 2c:44:fd:8c:e9:ef brd ff:ff:ff:ff:ff:ff
```

And my initial Networkmanager configuration looks like below:
```bash
root@box ~# nmcli  connection
NAME      UUID                                  TYPE      DEVICE
enp9s0f3  ca27e3fc-d80a-370f-97ce-cf4376e7af7e  ethernet  enp9s0f3
enp9s0f1  23b95567-e538-35e7-81e3-224436db0579  ethernet  enp9s0f1
enp9s0f2  3546d570-2c76-4c18-a472-ae420b8b4f30  ethernet  enp9s0f2
lo        f4669ca5-fea8-4856-a703-d02e46d72a5f  loopback  lo
enp7s0    d8e6dc21-34db-47bf-9686-769cbe4eabdb  ethernet  --
enp9s0f0  d283abf6-2821-4572-bcf7-64c18e89c6c4  ethernet  --
```

Steps
=====
1. Create main bond interface
```bash
nmcli connection add type bond ifname bond0 bond.options "miimon=100,mode=balance-alb"
```
1. At this point the new bond interface should be create but it's a bond with no physical NICs plugged yet.
```bash
root@box ~# ip a s dev bond0
9: bond0: <NO-CARRIER,BROADCAST,MULTICAST,MASTER,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether d6:9a:7c:62:71:b4 brd ff:ff:ff:ff:ff:ff
root@box ~# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v6.13.4-200.fc41.x86_64

Bonding Mode: adaptive load balancing
Primary Slave: None
Currently Active Slave: None
MII Status: down
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0
Peer Notification Delay (ms): 0
```
1. On top of existing physical network interfaces create _slave_ interfaces. In my example,
I will go crazy and create 5 slave interfaces :)
```bash
nmcli con add type ethernet ifname enp7s0 master bond0
nmcli con add type ethernet ifname enp9s0f0 master bond0
nmcli con add type ethernet ifname enp9s0f1 master bond0
nmcli con add type ethernet ifname enp9s0f2 master bond0
nmcli con add type ethernet ifname enp9s0f3 master bond0
```
These will create new _slave_ type interface for each corresponding physical interface,
these _slave_ interfaces are not _plugged_ to main bond interface yet.
```bash
root@box ~# nmcli connection | grep slave
bond-slave-enp9s0f0  2ad0aec5-7ac9-41b5-8283-dab4766740ae  ethernet  enp9s0f0
bond-slave-enp7s0    43c7837f-b883-49b5-8a79-314c9d0bb64e  ethernet  --
bond-slave-enp9s0f1  c0f99c24-7c6b-4f07-9ca8-2bd448397c6e  ethernet  --
bond-slave-enp9s0f2  1a4c22f8-4df9-4147-b33f-53ca81e5336e  ethernet  --
bond-slave-enp9s0f3  6fe6f5b1-2167-4ca3-b4e6-a80e449aa677  ethernet  --
```
1. Bring up all the interfaces
```bash
nmcli connection up bond-slave-enp9s0f0
nmcli connection up bond-slave-enp9s0f1
nmcli connection up bond-slave-enp9s0f2
nmcli connection up bond-slave-enp9s0f3
```
1. Configure IP address for new bond interface.
```bash
nmcli connection modify 2284f4cf-f25c-48e5-a805-c0838d3b1338 \
    connection.id bond0 \
    ipv4.method manual \
    ipv4.gateway 192.168.14.1 \
    ipv4.address 192.168.14.17/23 \
    ipv4.dns 192.168.14.14,192.168.14.1 \
    ipv4.dns-search kresy.local \
    ipv6.method manual \
    ipv6.dns fd00:f00c::ffff:c0a8:e0e \
    ipv6.dns-search kresy.local \
    ipv6.addresses fd00:f00c::ffff:c0a8:e11/64 \
    ipv6.gateway fd00:f00c::ffff:c0a8:e01
```

Site notes
==========
1. The bond mode which I configured for this example is _adaptive load balancing_
   or ALB mode. This mode allows to distribute your TX and RX traffic across all
   physical interfaces with no switch configuration.

