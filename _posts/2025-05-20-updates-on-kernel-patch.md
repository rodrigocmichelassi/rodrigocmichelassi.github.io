---
title: Updates on Kernel patch submission
date: 2025-05-20 21:30:00 +/-0300
categories: [Open Source Software Development, Kernel contributions]
description: "In this post, we will cover how the communication with the Kernel mantainers went and what is our patch submission status."
tags: [linux, kernel, kernel-linux, mac0470, open-source, i2c, iio, contribution]
---

On the last post, I have covered the Kernel contribution chosen by me and my partner, and explained a little about the return obtained by the Kernel mantainer. Going back on it, I feel like it would be nice to have a separate post to cover that communication process, because a lot can get lost on other posts.

### The patch answer

Our initial patch was proposing the creation of a new function in order to avoid code duplication and promote code readability. In a nutshell, our code change was actually proposing a function call that would just call 3 operations: the i2c xfer process, disabling i2c slave and checking i2c status, in order to return a positive value or raise an error.

What we failed to see in that patch is that those operations were always being called sequentially, and only on the file we have modified, so we did not need to create a new function to do so. Instead, we could only call the i2c slave disabling and status check inside the function that performs xfer. The mantainer said exactly that:

![Jean Baptiste response]({{ '/assets/img/2025-05-20-updates-on-kernel-patch/baptiste.png' | relative_url }})
_Jean Baptiste's answer to our patch_

The mantainer correctly identified our mistake and also gave us a suggestion on how to procceed with this patch. We also failed to understand another comment, that another Kernel mantainer is going to identify on a further response.

### Our last modifications

The following code is huge, but it is pretty much what we have just explained. We attributed i2c slave disabling and status check on the master xfer function, in order to get a better readability on the code. The master xfer function follows:

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

### Response and communication

With that new patch sent and the error appointed corrected, it is time to see the new responses. We actually waited a little before sending again this new version of the contribution because of holidays. We got a message stating that Jean Baptiste's office would not be working for a couple of days and during the weekend, so we postponed the e-mail. As a matter of fact, we also tried to confirm with the mantainer if we understood the patch, but got no response.

While time was running by, we decided to try and apply his suggestions on our own, taking a risk and running it by the TA's, who also thought we were right on procceding the way we did. So we sent the new patch (V3, since we messed up on the V2 e-mail), and after some days, we got a response from Marcelo Schmitt, a Kernel mantainer who does not work exactly on this IIO driver, but is an USP alumnus, who we were sending copies from our patches for review. He is not really able to approve our patch, but sent this message:

![Marcelo Schmitt response]({{ '/assets/img/2025-05-20-updates-on-kernel-patch/schmitt.png' | relative_url }})
_Marcelo Schmitt's answer to our patch_

Based on his answer, we saw that we could have added a slave slot parameter on the function call, even though it would not make a huge difference. Also, another suggestion he made was a code styling one, on the variable declaration, but he just left it up for Baptiste to decide if another version was necessary.

After that, we spent some more days waiting for Baptiste's answer on this patch, to see if we were able to continue, but got no response whatsoever. A weird fact is that another mantainer tagged our e-mail, to see if he would answer:

![Jonathan Cameron response]({{ '/assets/img/2025-05-20-updates-on-kernel-patch/jonathan.png' | relative_url }})
_Jonathan Cameron's tagging our patch_

Now, there is nothing really we can do. We are just waiting for Baptiste's new response for this contribution, but hopefully he is going to take a look at it soon and this patch will be reviewed.

### Final updates

After all those responses and another e-mail from Jonathan Cameron asking for an approval on the patch, Jean-Baptiste finally came through and approved our patch!

![Jean's approval]({{ '/assets/img/2025-05-20-updates-on-kernel-patch/baptiste-final.png' | relative_url }})
_Final approval for our patch_

Now, we are just waiting for final updates on the Kernel contribution, but it has been finally approved and we will soon have our first ever code running on Kernel.