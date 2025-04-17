---
title: Refactoring I2C auxiliary transfer functions
date: 2025-04-16 20:30:00 +/-0300
categories: [Open Source Software Development, Kernel contributions]
tags: [linux, kernel, kernel-linux, setup, ARM, boot, mac0470, floss, open-source, i2c, iio]
---

Alongside [Isabella Caselli](https://isacaselli.github.io/), I have just sent my first contribution to the Linux Kernel, on the IIO Tree. This contribution was done on the `MPU6050`, which is an IMU with accelerometer and gyroscope, using the I2C Master Mode. The I2C is a protocol for communication between sensors and a processor.

The code we have altered can be found at `iio/drivers/iio/imu/inv_mpu6050/inv_mpu_aux.c`, where the original problem was that the `inv_mpu_aux_write` and `inv_mpu_aux_read` functions had some repetitions on the code snippet shown below:

```c
/* do i2c xfer */
ret = inv_mpu_i2c_master_xfer(st);
if (ret)
        goto error_disable_i2c;

/* disable i2c slave */
ret = regmap_write(st->map, INV_MPU6050_REG_I2C_SLV_CTRL(0), 0);
if (ret)
        goto error_disable_i2c;

/* check i2c status */
ret = regmap_read(st->map, INV_MPU6050_REG_I2C_MST_STATUS, &status);
if (ret)
        return ret;
if (status & INV_MPU6050_BIT_I2C_SLV0_NACK)
        return -EIO;

error_disable_i2c:
        regmap_write(st->map, INV_MPU6050_REG_I2C_SLV_CTRL(0), 0);
        return ret;
```

Our goal here was to refactor that repeated code, promoting readability and organization, putting it all inside the function `inv_mpu_aux_exec_xfer`. This function is responsible for communicating with an auxiliary sensor, start data transfer (i2c xfer), disable slave (external device) and check if the communication went well. If not, the function returns an error.

Finally, with that moved, we had to make a function call where that code snippet happened, and handle the error return:

```c
int inv_mpu_aux_read(const struct inv_mpu6050_state *st, uint8_t addr,
                     uint8_t reg, uint8_t *val, size_t size)
{
    /* [...] previous code */
    ret = inv_mpu_aux_exec_xfer(st);
    if(ret)
        return ret;

    /* read data in registers */
    return regmap_bulk_read(st->map, INV_MPU6050_REG_EXT_SENS_DATA,
                            val, size);
}

int inv_mpu_aux_write(const struct inv_mpu6050_state *st, uint8_t addr,
                      uint8_t reg, uint8_t val)
{
    /* [...] previous code */
    ret = inv_mpu_aux_exec_xfer(st);
    if(ret)
        return ret;

    return 0;
}
```

Even though this was a small contribution, it felt really great to start having a place as a linux kernel contributor, and it sure helps understanding the kernel structure and patch sending pipeline, while still promoting code readability for future contributors. Even with a small work, we can do something great for the community.

Hopefully, this contribution gets accepted, and this page gets updated :)