---
title: Linux kernel build configuration and modules
date: 2025-03-31 13:45:00 +/-0300
categories: [Open Source Software Development, Linux Kernel Environment Setup]
tags: [linux, kernel, kernel-linux, setup, ARM, boot, mac0470]
---

In this post, I will be covering my experience following FLUSP tutorial 3, on how to [make new kernel modules and create build configurations](https://flusp.ime.usp.br/kernel/modules-intro/) for new kernel features.

### Errors obtained

This one tutorial was really tricky for me. I got some errors that I did not quite understand, and the TA's were really the ones to solve and explain those to me. Creating the module and setting it to the .config file was actually not hard. All those steps were easily solved.

The real problem came when installing the new kernel modules. When running the VM, by typing ```lsmod``` or ```modinfo mod_name``` I pretty much got no response, for most of the time. After re-running the tutorial, I was able to list the modules with ```lsmod```, but still no answer running modinfo. The error was happening when running ```make modules_install```, because of a missing directory. In that case, when compiling the modules, those were not being passed to the VM.

Solving that step took a long time, and also got me doing the tutorials all over again.

This one tutorial was also really important in order to understand some concepts that I had not understood before.