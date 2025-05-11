---
title: Refactoring I2C auxiliary transfer functions
date: 2025-04-16 20:30:00 +/-0300
categories: [Open Source Software Development, Kernel contributions]
tags: [linux, kernel, kernel-linux, setup, ARM, boot, mac0470, floss, open-source, i2c, iio]
---

Alongside [Isabella Caselli](https://isacaselli.github.io/), I have just sent my first contribution to the Linux Kernel, on the IIO Tree. This contribution was done on the `MPU6050`, which is an IMU with accelerometer and gyroscope, using the I2C Master Mode. The I2C is a protocol for communication between sensors and a processor.

## The contribution [V1]

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

## Feedback and changes

If it interests anyone to see, our patch can be found at the [lore platform](https://lore.kernel.org/linux-iio/20250428132551.176788-1-bellacaselli20@gmail.com/). After sending our contribution to the Kernel mailing list, we got a response from Jean-Baptiste Maneyrol, an IIO mantainer, where he stated that creating a new function to avoid the code duplication may make the code readability not the best, and, on his words, "We already have a common function `inv_mpu_i2c_master_xfer()` for handling master I2C transfer".

That being said, we went back to the code, following his advice, and realized that we did not read the code carefully enough, since not only `inv_mpu_i2c_master_xfer()` is only called on the read and write functions we had previously tried to improve, but also is always followed by the disabling of i2c slave and i2c status check. By having that information now, it hit us that we could just move these two operations to the master xfer function, and handle a little bit more information there. The new master xfer function follows:

```c
static int inv_mpu_i2c_master_xfer(const struct inv_mpu6050_state *st)
{
        /* use 50hz frequency for xfer */
	const unsigned int freq = 50;
	const unsigned int period_ms = 1000 / freq;
	uint8_t d;
	unsigned int user_ctrl;
	int ret;
	unsigned int status;

	/* set sample rate */
	d = INV_MPU6050_FIFO_RATE_TO_DIVIDER(freq);
	ret = regmap_write(st->map, st->reg->sample_rate_div, d);
	if (ret)
		return ret;

	/* start i2c master */
	user_ctrl = st->chip_config.user_ctrl | INV_MPU6050_BIT_I2C_MST_EN;
	ret = regmap_write(st->map, st->reg->user_ctrl, user_ctrl);
	if (ret)
		goto error_restore_rate;

	/* wait for xfer: 1 period + half-period margin */
	msleep(period_ms + period_ms / 2);

	/* stop i2c master */
	user_ctrl = st->chip_config.user_ctrl;
	ret = regmap_write(st->map, st->reg->user_ctrl, user_ctrl);
	if (ret)
		goto error_stop_i2c;

	/* restore sample rate */
	d = st->chip_config.divider;
	ret = regmap_write(st->map, st->reg->sample_rate_div, d);
	if (ret)
		goto error_restore_rate;

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

	return 0;

error_stop_i2c:
	regmap_write(st->map, st->reg->user_ctrl, st->chip_config.user_ctrl);
error_restore_rate:
	regmap_write(st->map, st->reg->sample_rate_div, st->chip_config.divider);
error_disable_i2c:
	regmap_write(st->map, INV_MPU6050_REG_I2C_SLV_CTRL(0), 0);
	return ret;
}
```

Now, we have just sent the [new code](https://lore.kernel.org/linux-iio/20250507184539.54658-1-bellacaselli20@gmail.com/) to review and hopefully now it will be accepted, and we will have our first Kernel contribution ever.