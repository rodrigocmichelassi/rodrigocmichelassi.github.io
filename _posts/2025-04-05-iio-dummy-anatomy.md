---
title: The IIO simple dummy anatomy
date: 2025-04-05 21:00:00 +/-0300
categories: [Open Source Software Development, Linux Kernel]
tags: [linux, kernel, kernel-linux, setup, ARM, boot, mac0470]
---

Following now the 5th tutorial of our series, we run the [IIO dummy anatomy tutorial](https://flusp.ime.usp.br/iio/iio-dummy-anatomy/). For this one, some really important concepts on how device drivers are actually coded and organized within the linux kernel are shown, and finally we get some C code action going on.

Even though this tutorial has no kernel setup to be followed, unlike the last ones, I got the feeling that it was one of the most important ones, as it introduces the notion needed for contributing to the IIO Tree, and also reveals where some other concepts we were reading about before are actually being used. Since we got a really big series of tutorials, and all the subjects were pretty much new to me, I found that I will need some time to adjust, so after reading this tutorial, I have decided to take a step back and review everything we have done so far.

## Important concepts

### Dummy channels setup

Before taking a review step back, I find if to be useful taking some notes about some things mentioned.

First of all, presented some tutorials ago, the Kernel runs with lots of **modules**, which are, basically, code snippets that can be inserted or removed from the Kernel as needed, and a clear example of that are the Devices Driver Modules, enabling Kernel communication with hardware equipments. We saw how to configure those modules and boot the Kernel with it previously, and our focus does not rely on that right now. On the other hand, it is necessary to have a basic knowledge on what the modules actually are.

Diving into the 5th tutorial, we get to see the Channels Setup for the IIO dummy driver. A channel is described as the _representation of a data channel_, that is, how data from different sensors/devices are organized and represented within code. Some devices may even have more than one channel (eg. the Accelerometer, that has 3 channels, representing the acceleration values from the X, Y and Z axes). We can setup channels using the ```iio_chan_spec``` struct.

The tutorial goes on to the configuration of the <code>iio dummy</code>, explaining some of the channels that are actually created, in order to simulate some sensors. I feel like there is just a lot to understand from it, but the code can be read and consulted at all times. 

One of the most important concepts to show up in here is the ```.type``` parameter, that represents the type of the data collected by the channel, and the <code>iio dummy</code> uses the <code>IIO_VOLTAGE</code> and <code>IIO_ACCEL</code> to represent the types of data, coming from a voltimeter and an accelerometer. Two other important concepts that are a part of this configuration are the ```.indexed```, that has value 1 if the channel is indexed, and ```.channel``` representing that index. Those are important to distinguish the channels, when a device has more than one data channel. 

Following up, ```.info_mask_separate``` designates attributes specific for the specific data channel we are configuring. We move on to the ```.scan_index```, that is a unique value, and indicates if buffer capture is enabled for the channel (with -1, when it is not) and ```scan_type```, representing the buffer data description itself.

Most of the things that appear in the other <code>iio dummy</code> channels come from the same concepts.

### <code>iio_dummy_read_raw</code> and <code>iio_dummy_write_raw</code> functions

The tutorial presents a huge amount of C code, with variable names that seem difficult to understand at first. 

In general, the read raw function goes through the correct channel and, depending on the type of the channel, presents the logic to read the channel values in the ```*val``` pointer. This variable receives the value provided by the ```iio_dummy_state``` struct itself.

For the write raw function, there is a similar setup, but now we receive a ```val``` variable and write it to the dummy state variable, instead of reading from it.

### Probe function

Some parts of this description were pretty much unclear to me, but the main idea was not that hard to figure out. The probe function is pretty much responsible for joining all the steps we have followed in one.

For a first operation, we allocate memory for an IIO Device, enough to store the ```iio_dummy_state```. With that done, we follow up to handling data initialization. As a final step, we register the device, and now we are pretty much done.

Now, we got a full look on the anatomy of a dummy device for IIO development.