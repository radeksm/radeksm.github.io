---
layout: post
title: WiFi configuration on Raspberry Pi
tags:
    - Raspberry Pi
    - IoT
author: Radosław Śmigielski
date: 2025-11-20 00:00:01 -0000
---

Why?
====
WiFi configuration of Raspberry Pi may be pain in the bad. Here is step by step guide with some examples.
This configuration is dedicated to build-in WiFi interface.

Environment
===========
1. Raspberry Pi 3 Model B Rev 1.2.
1. Raspberry Pi OS Lite (64 bit) operating system (no GUI, only cmd).

Symptoms
========
1. Initial state after fresh installaton.
* WiFi is being disabled at the software level by NetworkManager:
```
root@pi15 /h/radek# rfkill list wifi
0: phy0: Wireless LAN
        Soft blocked: yes
        Hard blocked: no
root@pi15 /h/radek# nmcli  radio wifi
disabled
```
* WiFi interface present but in DOWN state
```
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether b8:27:eb:ad:b7:60 brd ff:ff:ff:ff:ff:ff
```
* NetworkManager configuration
```
root@pi15 /h/radek# grep WirelessEnabled /var/lib/NetworkManager/NetworkManager.state
WirelessEnabled=false
```
* WPA supplicant refuse to start
```
root@pi15 /h/radek# wpa_supplicant  -i  wlan0 -c  /etc/wpa_supplicant/wpa_supplicant.conf
Successfully initialized wpa_supplicant
rfkill: WLAN soft blocked
rfkill: WLAN soft blocked
```

HowTo
=====
Below steps use NetworkManager which is enabled by default in recent systems.

1. Enable WiFi using NetworkManager
```
root@pi15 /h/radek# nmcli radio wifi on
root@pi15 /h/radek# nmcli radio wifi
enabled
```
1. Check WiFi state with rfkill
```
root@pi15 /h/radek# rfkill  list wifi
1: phy0: Wireless LAN
        Soft blocked: no
        Hard blocked: no
```
1. Configure WPA supplicant, create */etc/wpa_supplicant/wpa_supplicant.conf* config file:
```
root@pi15 /root# cat /etc/wpa_supplicant/wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=PL
network={
    ssid="wifi-a"
    psk="My%SecretX2"
    priority=10
}
network={
    ssid="wifi-b"
    psk="My%SecretX2"
    priority=20
}
network={
    ssid="wifi-c"
    psk="My%SecretX2"
    priority=30
}
```
1. Make sure *wpa_supplicant* is NOT running and test it as root first:
```
systemctl  stop wpa_supplicant.service
```
1. Most common problem, wrong password:
```
root@pi15 /root# wpa_supplicant  -i  wlan0 -c  /etc/wpa_supplicant/wpa_supplicant.conf
...
wlan0: Trying to associate with 8a:7c:61:c2:e2:c9 (SSID='wifi-a' freq=2417 MHz)
wlan0: Associated with 8a:7c:61:c2:e2:c9
MBO: Disable MBO/OCE due to misbehaving AP not having enabled PMF
wlan0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
wlan0: CTRL-EVENT-REGDOM-CHANGE init=COUNTRY_IE type=COUNTRY alpha2=DE
wlan0: CTRL-EVENT-DISCONNECTED bssid=8a:7c:61:c2:e2:c9 reason=15
wlan0: WPA: 4-Way Handshake failed - pre-shared key may be incorrect
wlan0: CTRL-EVENT-SSID-TEMP-DISABLED id=0 ssid="wifi-a" auth_failures=1 duration=10 reason=WRONG_KEY
...
```
1. Restart and enable *wpa_supplicant.service*:
```
systemctl  enable --now  wpa_supplicant.service
```
1. Add WiFi connection using NetworkManager:
```
nmcli connection add type wifi con-name "wifi-a" ifname wlan0 ssid "wifi-a" wifi-sec.key-mgmt wpa-psk wifi-sec.psk ""My%SecretX2
```
1. Finally configure IP of your new shiny WiFi interface:
```
nmcli connection modify <UUID_OF_WIFI_CONNECTION> \
    ipv4.method manual \
    ipv4.dns 192.168.14.1 \
    ipv4.dns-search kresy.local \
    ipv4.addresses 192.168.14.115/23  \
    ipv4.gateway 192.168.14.1
nmcli connection modify <UUID_OF_WIFI_CONNECTION> \
    ipv6.method manual \
    ipv6.dns fd00:f00c::ffff:c0a8:e01 \
    ipv6.dns-search kresy.local \
    ipv6.addresses fd00:f00c::ffff:c0a8:e73/64 \
    ipv6.gateway fd00:f00c::ffff:c0a8:e01
```

**Enjoy!**


