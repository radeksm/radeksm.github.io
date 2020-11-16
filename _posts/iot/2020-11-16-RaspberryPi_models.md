---
layout: post
title: Raspberry Pi model discovery
tags:
    - Raspberry Pi
    - IoT
author: Radosław Śmigielski
---

Raspberry Pi have model and version printed on top of circuit board.
But what if your Pi is somewhere far away from you or you don't have
access to it? There is an easy way to discover model of your Pi remotely,
assuming you can ssh to it.

Raspberry Pi model is reported by kernel at the early stage of the boot process.
```
sudo dmesg | grep model
```

Output examples:
```
[    0.000000] OF: fdt:Machine model: Raspberry Pi Model B Plus Rev 1.2
[    0.000000] OF: fdt: Machine model: Raspberry Pi 3 Model B Rev 1.2
```
