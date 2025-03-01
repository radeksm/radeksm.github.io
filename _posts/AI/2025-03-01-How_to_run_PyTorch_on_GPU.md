---
layout: post
title: How can I run any code on my GPU, instead on CPU?
tags:
    - CUDA
    - AI
author: Radosław Śmigielski
---

Why?
====
While I started to play with AI Large Language Models, I wanted to speed up the process
of training models on my oldish home PC running somewhere in my closet. Of course training
models on CPU is not the most efficient way of doing it, I wanted to use my GPU.
The things worked rather slow for me and I started asking myself a question if my models
run on CPU or GPU? And how can I run any code on my GPU, instead on CPU?

Hardware
========
For the purpose of this exercise I re-activated some old PC.
* Everything that I describe below, I ran on Linux (Fedora 41).
* My CPU is AMD Ryzen 7 5700G.
* My GPU is old NVIDIA® GeForce® GT 1030

Intro
=====
Let's start by checking if our CUDA works fine and if it can see our GPU.
This is not always that straight forward and may give you some headache but this is a topic
for a whole another different story.  
So let's start from something very simple, NVIDIA CUDA provides __nvidia-smi__
tool to check your GPU presence and status.

This is how your idle GPU should looks like:
```
ai@box ~/s/r/_/AI > nvidia-smi
Sat Mar  1 06:33:20 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.86.15              Driver Version: 570.86.15      CUDA Version: 12.8     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce GT 1030         Off |   00000000:01:00.0 Off |                  N/A |
| 35%   26C    P8             N/A /   19W |       4MiB /   2048MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```


