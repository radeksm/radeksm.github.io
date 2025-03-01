---
layout: post
title: How can I run any code on my GPU, instead on CPU?
tags:
    - AI
    - CUDA
    - PyTorch
author: Radosław Śmigielski
---

Why?
====
While I started to play with AI Large Language Models, I wanted to speed up the process
of training models on my oldish home PC running somewhere in my closet. Of course training
models on CPU is not the most efficient way of doing it, I wanted to use my GPU.
The things worked rather slow for me and I started asking myself a question if my models
run on CPU or GPU? And how can I run any code on my GPU, instead on CPU?

Envrironment
============
For the purpose of this exercise I re-activated some old PC.
* Everything that I describe below, I ran on Linux (Fedora 41).
* My CPU is AMD Ryzen 7 5700G.
* My GPU is old NVIDIA® GeForce® GT 1030
All Python components were installed inside Python 3.x virtual environment.
```
python -m venv ~/venv-cuda --upgrade-deps --symlinks --system-site-packages
source ~/venv-cuda/bin/activate
```

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
Have a look on some of the interesting values like: __GPU-Util__, __Memory-Usage__, 
__Temp__, __Perf__ (preformance profile) and finally empty list of __Processes__ running on GPU.

PyTorch installation
--------------------
Let's start from PyTorch installation
```
source ~/venv-cuda/bin/activate
pip3 install torch torchvision torchaudio
```
\* do not skip _torchaudio_, even if you think you don't need it :)  

Now, we can check if PyTorch can detect and use your GPU:
```
import torch

torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.get_device_name(torch.cuda.current_device())
```

Python Torch code
-----------------
Python code example to run on CPU and GPU.
```python

import torch
from datetime import datetime

torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.get_device_name(torch.cuda.current_device())

size = 8192
  

def measure_time(func):
    def inner_func(hw):
        start = datetime.now()
        func(hw)
        end = datetime.now()
        td = (end - start).total_seconds()
        print(f"Run on {hw}. Time of execution: {td:.03f}[s]. Matrix size: {size}")
    return inner_func

@measure_time
def crunch_the_numbers(hw):
    print(f"Crunching numbers on {hw} . . .")
    a = torch.rand(size, size, device=hw, dtype=torch.float16)
    b = torch.rand(size, size, device=hw, dtype=torch.float16)
    c = torch.rand(size, size, device=hw, dtype=torch.float16)

    for x in range(size):
        y0 = a + b
        y1 = y0 + c
        y2 = y1 * a
        y3 = y2 - b
        y4 = y2 + y3
 

# ON CPU
crunch_the_numbers("cpu")
print(f"Done on CPU (size={size}).")

# ON GPU
crunch_the_numbers("cuda")
print(f"Done on GPU (size={size}).")
```

