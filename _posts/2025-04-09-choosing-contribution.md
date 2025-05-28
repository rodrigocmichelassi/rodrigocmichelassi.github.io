---
title: Choosing a Kernel contribution
date: 2025-04-09 17:00:00 +/-0300
categories: [Open Source Software Development, Kernel contributions]
tags: [linux, kernel, kernel-linux, mac0470, floss, open-source]
---

After spending some weeks setting up a Linux Kernel development environment, we are finally being able to start contributing to the Kernel. For that, the TA's, who are way more experienced on contributing to the Kernel, have worked on finding some possible contributions, that are not that hard for beginners.

By having some discussions with Isabella, who I'm working with, we have decided not to take the easiest contributions (such as solving code styling warnings) and actually put a hand on actual Kernel code. That decision is based upon the fact that, even though our contributions are not the hardest (I hope), we still get to understand a little more about how the whole structure is handled and organized, and also get a hand on the code organization.

I'm hoping to blog more about how the contributions are going, and hopefully get a patch accepted on the Kernel, in the next couple of weeks. Even if it fails, everything will be documented.

### First contribution

This one is called: Claim IIO direct mode to avoid interleave register read/write with buffered data capture.

There is a huge description, but the general idea is to use the ```iio_device_claim_direct()``` function to lock device mode, to stop concurrent access to the buffer, and after that adding ```iio_device_release_direct()``` to re-allow that access to users. We have chosen to do that on the ```drivers/iio/light/isl29125.c``` file, on the light driver.

If all goes right, we will be able to understand where this change is needed on the file, study the code and think about where to add that.

### Second contribution

For this contribution, there is no official name, but we are actually working on the Inercial Measurement Unit (IMU) driver, on the IIO tree. Although this contribution is a little less technical, it feels way more interesting. 

If anyone bothers to take a look at the ```imu/inv_mpu6050/inv_mpu_aux.c::inv_mpu_aux_read``` and the ```imu/inv_mpu6050/inv_mpu_aux.c::inv_mpu_aux_write``` functions, before our contribution, they may be able to see that those functions have their differences, but the second part of the i2c management is pretty much the same.

The idea for this contribution is to simply identify the simmilar parts for both functions and refactor the code, adding a new function call on both to perform the repeated activity. That way, a simple difference on the code readability may help future contributors and optimize the Kernel code quality.