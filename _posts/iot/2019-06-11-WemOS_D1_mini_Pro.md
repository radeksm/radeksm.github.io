---
layout: post
title: D1 mini Pro random notes
tags:
    - WemOS
    - IoT
author: Radosław Śmigielski
---
At the time of writing this WemOS D1 mini Pro (v1.0.0) has been already retired
but it's still be popular board and widely available for purchase.

Power consumption
-----------------

WemOS D1 mini Pro cofiguration:
1. directly powered by 3.3V (no USB)
2. LED_BUILTIN off
3. Serial console on
4. CPU speed 80 MHz

|Description|Current [mA]|
|:-:|:-:|
|WiFi off (WiFi.forceSleepBegin(); delay(_loong_))| 14.6 |
|WiFi off (WiFi.forceSleepBegin())| 18.5 |
|Connected to WiFi (802.11b), idle connection| 71-72  |
|   |   |
|   |   |
|   |   |
|   |   |
