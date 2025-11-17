---
layout: post
title: "How to extend life of your laptop battery?"
tags:
    - Linux
author: Radosław Śmigielski
date: 2025-11-17 00:00:01 -0000
---

Why?
====
Most of the laptops' batteries are lithium-ion battery which don't like to be overcharge or deep discharge.
So what if your laptop is living on AC all day long? Up until kernel XXXX there was nothing you could so about it.
But the things have changed and now there are two kernel params which helps you keep your battery change level
within specific limits.

Environment
===========
ThinkPad laptops.

How ?
=====
Set the chargining limits manually:
```bash
echo 40 > /sys/class/power_supply/BAT*/charge_control_start_threshold
echo 80 > /sys/class/power_supply/BAT*/charge_control_end_threshold
```

Or using script:
```bash
#!/bin/bash

# Battery device path
BATTERY_PATH="/sys/class/power_supply/BAT*"

# Paths to battery threshold files
START_THRESHOLD="${BATTERY_PATH}/charge_control_start_threshold"
END_THRESHOLD="${BATTERY_PATH}/charge_control_end_threshold"

# Check if running as root
check_root() {
    if [ "$EUID" -ne 0 ]; then
        echo "Error: This script must be run as root (use sudo)"
        exit 1
    fi
}

start() {
    echo "Enabling battery charge limits (40-80%)..."
    echo 40 > "$START_THRESHOLD"
    echo 80 > "$END_THRESHOLD"
    echo "Battery charge limits enabled."
}

stop() {
    echo "Disabling battery charge limits (0-100%)..."
    echo 0 > "$START_THRESHOLD"
    echo 100 > "$END_THRESHOLD"
    echo "Battery charge limits disabled."
}

status() {
    echo "Current battery charge thresholds:"
    echo "  Start: $(cat $START_THRESHOLD)%"
    echo "  End: $(cat $END_THRESHOLD)%"
}

case "$1" in
    start)
        check_root
        start
        ;;
    stop)
        check_root
        stop
        ;;
    status)
        status
        ;;
    *)
        echo "Usage: $0 {start|stop|status}"
        echo "  start  - Enable battery charge limits (40-80%)"
        echo "  stop   - Disable battery charge limits (0-100%)"
        echo "  status - Show current charge thresholds"
        exit 1
        ;;
esac
```


Site notes
==========
TBD.

