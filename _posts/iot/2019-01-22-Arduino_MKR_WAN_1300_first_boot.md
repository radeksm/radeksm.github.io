---
layout: post
title: Arduino MKR WAN 1300 first boot
tags:
    - Arduino
    - IoT
author: Radosław Śmigielski
---
I've just got my new Arduino MKR WAN 1300.

![Arduino MKR WAN 1300](https://raw.githubusercontent.com/radeksm/r-blobs/master/radeksm.github.io/iot/Arduino_MKR_WAN_1300.jpg){: height="400px" width="300px"}

And after I connected it to my Linux laptop there was no USB/serial device
and *dmesg* showed me below errors:
{% highlight bash %}
[1209022.008060] usb 1-2: new full-speed USB device number 88 using xhci_hcd
[1209022.122104] usb 1-2: device descriptor read/64, error -71
[1209022.339124] usb 1-2: device descriptor read/64, error -71
[1209022.556097] usb 1-2: new full-speed USB device number 89 using xhci_hcd
[1209022.670124] usb 1-2: device descriptor read/64, error -71
[1209022.891087] usb 1-2: device descriptor read/64, error -71
[1209022.993237] usb usb1-port2: attempt power cycle
[1209023.621015] usb 1-2: new full-speed USB device number 90 using xhci_hcd
[1209023.621166] usb 1-2: Device not responding to setup address.
[1209023.825114] usb 1-2: Device not responding to setup address.
[1209024.032992] usb 1-2: device not accepting address 90, error -71
[1209024.147037] usb 1-2: new full-speed USB device number 91 using xhci_hcd
[1209024.147261] usb 1-2: Device not responding to setup address.
[1209024.353273] usb 1-2: Device not responding to setup address.
[1209024.561329] usb 1-2: device not accepting address 91, error -71
[1209024.561472] usb usb1-port2: unable to enumerate USB device

[1214456.794042] usb usb1-port1: attempt power cycle
[1214457.837935] usb 1-1: new full-speed USB device number 82 using xhci_hcd
[1214457.838180] usb 1-1: Device not responding to setup address.
[1214458.041983] usb 1-1: Device not responding to setup address.
[1214458.249836] usb 1-1: device not accepting address 82, error -71
[1214459.145945] usb usb1-port1: Cannot enable. Maybe the USB cable is bad?
[1214459.146042] usb usb1-port1: unable to enumerate USB device
[1214468.049688] Bluetooth: hci0: last event is not cmd complete (0x0f)

{% endhighlight %}

There is not much similar problems reported around but on one of them someone adviced
to double press reset button. I didn't really work for but I tried few other crayzy
combinations:
1. keep pressing reset button
1. press and hold reset button
1. double press reset button

All of them with power connected to my board.

And finally I got this:
{% highlight bash %}
[root@photon]# lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 04f2:b51c Chicony Electronics Co., Ltd 
Bus 001 Device 005: ID 138a:003f Validity Sensors, Inc. VFS495 Fingerprint Reader
Bus 001 Device 004: ID 8087:0a2b Intel Corp. 
Bus 001 Device 112: ID 2341:8053 Arduino SA 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub


[1209053.334111] usb 1-2: new full-speed USB device number 92 using xhci_hcd
[1209053.467739] usb 1-2: New USB device found, idVendor=2341, idProduct=8053, bcdDevice= 1.00
[1209053.467744] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[1209053.467746] usb 1-2: Product: Arduino MKR WAN 1300
[1209053.467748] usb 1-2: Manufacturer: Arduino LLC
[1209053.467750] usb 1-2: SerialNumber: B8960F6A504E4B36322E314AFF052121
[1209053.501721] cdc_acm 1-2:1.0: ttyACM0: USB ACM device
[1209053.502705] usbcore: registered new interface driver cdc_acm
[1209053.502709] cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
{% endhighlight %}
