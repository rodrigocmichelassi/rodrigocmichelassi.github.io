---
title: Linux Kernel Character Device Drivers
date: 2025-03-31 21:30:00 +/-0300
categories: [Open Source Software Development, Linux Kernel Environment Setup]
tags: [linux, kernel, kernel-linux, setup, ARM, boot, mac0470]
---

This post is referring to the last linux kernel setup on the series of FLUSP tutorials, explaining aspects of [Linux character devices](https://flusp.ime.usp.br/kernel/char-drivers-intro/). On this specific tutorial, we were not requested to execute many commands, but instead read and understand general concepts. With that in mind, we had no errors whatsoever.

## Concepts


### Device character Drivers

Character device drivers are ways to define how hardware and software should operate together, managing data flow and access. It is an abstraction layer, enabling communication between applications and the underlying system (the OS). In a more specific way, a character device is used to support devices that can be read from or written to, with small data transfers, usually byte size or few bytes size.

### Device ID

On your default linux OS, you can find character devices under ```/dev```. Usually, every file associated with character devices carry a major and minor numbers. The device drivers actually handle syscalls for character device files, and both are associated with a device ID, given by the major and minor number. The ID is actually a 32-bit unsigned integer, defined by the dev_t type, having the 12 most significant bits being the major device number, while the 20 other bits store the minor number.

Nowadays, it is possible to store different drivers with the same major number, as long as the range of their respective minor numbers do not interesect. You can obtain the major number of different character devices by simply typing ```cat /proc/devices``` on your linux terminal, and check the name of each character devices. You can also run the ```stat /dev/character_device``` command, in order to get major and minor number.

### File Operations

In order to configure character device drivers, we can use a ```struct file_operations``` object to handle syscalls. In the tutorial, we defined syscalls related to basic operations, such as read, write, open and close.

We use those structs to tie together character IDs and file operations, in order to get an actual character device ```struct cdev```.

For the rest of the tutorial, we have set up a simple character device driver and tested its functionality. Everything went smoothly.