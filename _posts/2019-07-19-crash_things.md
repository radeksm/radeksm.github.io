---
layout: post
title: How to crash things?
tags:
    - crash
author: Radosław Śmigielski
---

Why?
====
Why do you want to crash your application or system? I don't know but I can help.

Crashes
=======
1. Simple C crash on SIGFPE, float division by zero.
   - code
   ```
   #include <stdio.h>
   main(){ return 1/0; }
   ```
   - stack trace
   ```
   #0  0x0000000000401115 n/a (/home/radek/src/a.out)
   #1  0x00007fcb7b2d3413 __libc_start_main (libc.so.6)
   #2  0x000000000040104e n/a (/home/radek/src/a.out)
   ```
1. Python 3.7 crashes when PYTHONHOME set to place where Python
   is not installed, signal ABRT.
   - code
   ```
   $ PYTHONHOME="/tmp" python3
   Fatal Python error: initfsencoding: Unable to get the locale encoding
   ModuleNotFoundError: No module named 'encodings'
   Current thread 0x00007ff04e190740 (most recent call first):
   Aborted (core dumped)
   ```
   - stack trace
   ```
   Stack trace of thread 14856:
   #0  0x00007ff04e1cb57f raise (libc.so.6)
   #1  0x00007ff04e1b5895 abort (libc.so.6)
   #2  0x00007ff04e5b8bb7 n/a (libpython3.7m.so.1.0)
   #3  0x00007ff04e62314b _Py_FatalInitError (libpython3.7m.so.1.0)
   #4  0x00007ff04e626bcb n/a (libpython3.7m.so.1.0)
   #5  0x00007ff04e78620c n/a (libpython3.7m.so.1.0)
   #6  0x00007ff04e78676c _Py_UnixMain (libpython3.7m.so.1.0)
   #7  0x00007ff04e1b7413 __libc_start_main (libc.so.6)
   ```
