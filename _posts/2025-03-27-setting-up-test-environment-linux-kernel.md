---
title: Setting up a test environment for Linux Kernel
date: 2025-03-27 21:30:00 +/-0300
categories: [Open Source Software Development, Linux Kernel Environment Setup]
tags: [linux, kernel, kernel-linux, setup, qemu, mac0470]
---

As a first step into open source software development, we must set configurations on our environment. For this, FLUSP has designed a series of tutorials in order to embrace new people to open source software development community, and helping them configure their environments. In this post, we will be [setting up a test environment using qemu and libvirt](https://flusp.ime.usp.br/kernel/qemu-libvirt-setup/), aiming to contribute to the Linux Kernel.

### Failed attempts

For the first time trying to follow this tutorial, since I do not own a computer with linux in it, I tried running everything on a Lima VM. So the whole point was setting up a VM inside a VM, and, as it turns out, it did not work. I was able to launch a VM, but not to create one with virsh. My guess is that all the problems I had were happening because my Lima VM did not have enough memory allocated, but it sure was a long shot.

A second failed attempt happened when I tried running everything remotely, using an AWS EC2 machine, with ubuntu or debian on it. They both failed at a different point.

### Successful attempt

Lucky as I am, even the successful attempt came with a series of errors, but I was able to solve it. For the class MAC0470 | Free and Open Source Software Development, the professor and TA's have decided to keep a machine physically running at the lab, so people who own macbooks and computers that are not really running linux could follow the tutorials and have a proper computer, to access remotely, and be able to work on linux kernel.

With that said, I was able to successfully create a virsh VM, get its information and also set up a SSH access, from the host to the VM.