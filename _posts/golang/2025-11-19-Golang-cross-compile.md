---
layout: post
title: "Cross compile Golang for your Raspberry Pi and other platforms"
tags:
    - Linux
author: Radosław Śmigielski
date: 2025-11-19 00:00:01 -0000
---

Why?
====
You probably have your own reason(s) why you need to compile Go code for
Raspberry Pi. Maybe you have own project which does not come in binary
form from your Raspberry Pi operating system (Raspbian, Ubuntu).
In my case, my old Raspberry Pi was too slow to compile Golang.

Environment
===========
Raspbian GNU/Linux 12 (bookworm) on Raspberry Pi Model B Plus Rev 1.2.

Golang cross compilaton
=======================
Golang cross comilation is the simplest way of cross compiling code,
I've ever seen so far ;)
It was first added in Golang 1.14 and has become usable in 1.15.
It allows to select your target by setting these two variables before
you compile your code:
1. GOOS - target operating system
1. GOARCH - target compilation architecture

GOOS available options:
1. android
1. darwin
1. dragonfly
1. freebsd
1. illumos\* - OpenSolaris fork
1. ios
1. js - experimental WebAssembly
1. linux
1. netbsd
1. openbsd
1. plan9
1. solaris
1. wasip1\*\* - WebAssembly System Interface
1. windows

_wasip1_\*) https://github.com/golang/go/issues/58141, https://wasi.dev  
_illumos_\*\*) OpenSolaris fork developed since 2010, after Oracle decided to discontinue OpenSolaris.


GOARCH available options:
1. 386
1. amd64
1. arm
1. arm64
1. loong64
1. mips
1. mips64
1. mips64le
1. mipsle
1. ppc64
1. ppc64le
1. riscv64
1. s390x
1. wasm

(!) Not all the combinations of GOARCH and GOOS are supported.
Supported combinations can be checked using this command:
```bash
go tool dist list
```
And this is how it looks on my Golang go1.24.10:
```
aix/ppc64
android/386
android/amd64
android/arm
android/arm64
darwin/amd64
darwin/arm64
dragonfly/amd64
freebsd/386
freebsd/amd64
freebsd/arm
freebsd/arm64
freebsd/riscv64
illumos/amd64
ios/amd64
ios/arm64
js/wasm
linux/386
linux/amd64
linux/arm
linux/arm64
linux/loong64
linux/mips
linux/mips64
linux/mips64le
linux/mipsle
linux/ppc64
linux/ppc64le
linux/riscv64
linux/s390x
netbsd/386
netbsd/amd64
netbsd/arm
netbsd/arm64
openbsd/386
openbsd/amd64
openbsd/arm
openbsd/arm64
openbsd/ppc64
openbsd/riscv64
plan9/386
plan9/amd64
plan9/arm
solaris/amd64
wasip1/wasm
windows/386
windows/amd64
windows/arm64
```

How ?
=====
Finally. After this long into here is how you can compile Go code
for different versions of Raspberry Pi:
1. Check your Raspberry Pi version:
```bash
radek@pi14:~ $ arch
armv6l
```
In my case this is ARM v6, in your case may be different.
1. Set GOARCH and GOOS and compile:
```bash
GOOS=linux GOARCH=arm GOARM=6 go build run.go
```
1. Check ths produced binary
```bash
$ file run
run: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, BuildID[sha1]=51d8599ec532368d6923160997a87b4f42d0faba, with debug_info, not stripped
```

Sources
=======
1. Golang source code
1. Go all supported platforms definition: _src/cmd/dist/build.go_

