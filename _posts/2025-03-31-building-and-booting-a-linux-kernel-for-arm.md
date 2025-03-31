---
title: Building and booting a custom Linux kernel for ARM
date: 2025-03-31 09:30:00 +/-0300
categories: [Open Source Software Development, Linux Kernel]
tags: [linux, kernel, kernel-linux, setup, ARM, boot, mac0470]
---

Following our sequence of setup posts for the Linux Kernel contribution environment, now we are following a [tutorial on how to compile the Linux Kernel and boot it](https://flusp.ime.usp.br/kernel/build-linux-for-arm/) in the VM that was set up in the previous post.

### Process and errors

First of all, we can not build and boot a kernel instance without having it on our machines. Since the whole kernel is a little too much, and this is the first time all of the students are developing something for the kernel, we are not cloning all of the linux trees. Our goal is to contribute specifically on the IIO Tree, so that's the only one we are cloning into our machine.

With that done, the whole thing was now about configuring everything, before compiling the kernel tree. In order to do so, we have generated the ```.config``` file and copy the whole IIO Tree, with the configuration file, into the VM we set up on the last post. At this point, I had some issues running ```make -C "$IIO_TREE" localmodconfig```, in order to minimize the amount of modules loaded on the configuration file. That happened because the make command was not finding the LSMOD variable, we have exported before. In order to fix that, I ran ```make LSMOD=vm_mod_list localmodconfig``` inside the $IIO_TREE directory.

Going beyond that point, I believe the rest of the tutorial went smoothly, we were able to compile the kernel instance, install the modules and boot our linux instance. We will talk more about modules on the next post.