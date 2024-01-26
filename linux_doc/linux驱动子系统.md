## LED子系统

### 设备树插件配置

参考soc\kernel\src\arch\arm64\boot\dts\rockchip\rk3568-lubancat-2.dtsi里的leds配置

soc\kernel\src\arch\arm64\boot\dts\rockchip\overlay新增rk3568-lubancat-virgo-overlay.dts文件，内容如下

soc\kernel\src\arch\arm64\boot\dts\rockchip\overlay\Makefile文件添加rk3568-lubancat-virgo-overlay.dtbo编译

~~~dtd
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/rockchip.h>

&leds {
	status = "okay";
	compatible = "gpio-leds";

	virgo_test_led: virgo-test-led {
		label = "virgo_test_led";
		default-state = "on";
		gpios = <&gpio0 RK_PB0 GPIO_ACTIVE_LOW>;
		pinctrl-names = "default";
		pinctrl-0 = <&virgo_test_led_pin>;
	};
};

&pinctrl {
	leds {
		virgo_test_led_pin: virgo-test-led-pin {
			rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
};

~~~



编译

~~~sh
root@79952becb170:/myvolume/soc/kernel/build# make device_tree
~~~

复制

~~~
产物soc\kernel\src\arch\arm64\boot\dts\rockchip\overlay\rk3568-lubancat-virgo-overlay.dtbo
复制到板子/boot/dtb/overlay
~~~

修改uEnv.txt文件增加如下内容
~~~
#virgo
dtoverlay=/dtb/overlay/rk3568-lubancat-virgo-overlay.dtbo
~~~

补充：uEnv.txt是一个软连接

~~~sh
root@lubancat:/boot/uEnv# ls -la
total 56
drwxrwxr-x 2 root root 4096 Sep 16 20:58 .
drwxr-xr-x 7 root root 4096 Sep 16 20:58 ..
lrwxrwxrwx 1 root root   20 Mar 28 01:54 uEnv.txt -> uEnvLubanCat2-V1.txt
~~~

### sys文件系统控制亮灭

~~~
root@lubancat:/sys/devices/platform/leds/leds/virgo_test_led# cat brightness 
255 
root@lubancat:/sys/devices/platform/leds/leds/virgo_test_led# echo 0 > brightness 
root@lubancat:/sys/devices/platform/leds/leds/virgo_test_led# echo 255 > brightness 
~~~



## INPUT子系统

### 设备树插件配置

在设备树目录搜索“gpio-keys”，模仿别人写的

~~~dtd
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/rockchip.h>
#include <dt-bindings/input/input.h>

/* 新增 */
&{/}{
	keys: gpio-keys {
		compatible = "gpio-keys";

		button {
			label = "button";
			gpios = <&gpio3 RK_PA5 GPIO_ACTIVE_LOW>;
			linux,code = <KEY_1>;
			pinctrl-names = "default";
			pinctrl-0 = <&button_pin>;
		};
	};
};

/* 插入 */
&leds {
	status = "okay";
	compatible = "gpio-leds";

	virgo_test_led: virgo-test-led {
		label = "virgo_test_led";
		default-state = "on";
		gpios = <&gpio0 RK_PB0 GPIO_ACTIVE_LOW>;
		pinctrl-names = "default";
		pinctrl-0 = <&virgo_test_led_pin>;
	};
};



&pinctrl {
	leds {
		virgo_test_led_pin: virgo-test-led-pin {
			rockchip,pins = <0 RK_PB0 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
	keys {
		button_pin: button-pin {
			rockchip,pins = <3 RK_PA5 RK_FUNC_GPIO &pcfg_pull_none>;
		};
	};
};


~~~



### 测试工具

~~~sh
cat@lubancat:~$ cat /proc/bus/input/devices 
I: Bus=0019 Vendor=0001 Product=0001 Version=0100
N: Name="gpio-keys"
P: Phys=gpio-keys/input0
S: Sysfs=/devices/platform/gpio-keys/input/input3
U: Uniq=
H: Handlers=kbd event3 cpufreq dmcfreq 
B: PROP=0
B: EV=3
B: KEY=4

cat@lubancat:~$ sudo hexdump /dev/input/event3 
0000000 4c1e 6506 0000 0000 99b6 000d 0000 0000
0000010 0001 0002 0000 0000 4c1e 6506 0000 0000
0000020 99b6 000d 0000 0000 0000 0000 0000 0000
0000030 4c1f 6506 0000 0000 cc76 0001 0000 0000
0000040 0001 0002 0001 0000 4c1f 6506 0000 0000
0000050 cc76 0001 0000 0000 0000 0000 0000 0000

cat@lubancat:~$ sudo evtest /dev/input/event3 
Input driver version is 1.0.1
Input device ID: bus 0x19 vendor 0x1 product 0x1 version 0x100
Input device name: "gpio-keys"
Supported events:
  Event type 0 (EV_SYN)
  Event type 1 (EV_KEY)
    Event code 2 (KEY_1)
Properties:
Testing ... (interrupt to exit)
Event: time 1694911457.911211, type 1 (EV_KEY), code 2 (KEY_1), value 0
Event: time 1694911457.911211, -------------- SYN_REPORT ------------
Event: time 1694911458.051194, type 1 (EV_KEY), code 2 (KEY_1), value 1
~~~



## IIC子系统

看设备树获取设备名字rk3399-i2c

~~~dtd
	i2c0: i2c@fdd40000 {
		compatible = "rockchip,rk3399-i2c";
		reg = <0x0 0xfdd40000 0x0 0x1000>;
		clocks = <&pmucru CLK_I2C0>, <&pmucru PCLK_I2C0>;
		clock-names = "i2c", "pclk";
		interrupts = <GIC_SPI 46 IRQ_TYPE_LEVEL_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&i2c0_xfer>;
		#address-cells = <1>;
		#size-cells = <0>;
		status = "disabled";
	};
~~~

找对应的驱动

~~~c
static const struct of_device_id rk3x_i2c_match[] = {
	//省略
	{
		.compatible = "rockchip,rk3399-i2c",
		.data = &rk3399_soc_data
	},
	{},
};
static struct platform_driver rk3x_i2c_driver = {
	.probe   = rk3x_i2c_probe,
	.remove  = rk3x_i2c_remove,
	.driver  = {
		.name  = "rk3x-i2c",
		.of_match_table = rk3x_i2c_match,
		.pm = &rk3x_i2c_pm_ops,
	},
};
~~~

重要结构体



~~~c
struct i2c_adapter {
	struct module *owner;
	unsigned int class;		  /* classes to allow probing for */
	const struct i2c_algorithm *algo; /* the algorithm to access the bus */
	void *algo_data;

	/* data fields that are valid for all devices	*/
	const struct i2c_lock_operations *lock_ops;
	struct rt_mutex bus_lock;
	struct rt_mutex mux_lock;

	int timeout;			/* in jiffies */
	int retries;
	struct device dev;		/* the adapter device */

	int nr;
	char name[48];
	struct completion dev_released;

	struct mutex userspace_clients_lock;
	struct list_head userspace_clients;

	struct i2c_bus_recovery_info *bus_recovery_info;
	const struct i2c_adapter_quirks *quirks;

	struct irq_domain *host_notify_domain;
};
~~~

i2c控制器

- `owner`：指向拥有该结构体的模块。
- `algo`：用于访问总线的算法。

- `algo_data`：与算法相关的数据。
- `retries`：尝试次数。
- `dev`：表示适配器设备的结构体。
- `name`：适配器名称



~~~c
struct i2c_algorithm {
	/* If an adapter algorithm can't do I2C-level access, set master_xfer
	   to NULL. If an adapter algorithm can do SMBus access, set
	   smbus_xfer. If set to NULL, the SMBus protocol is simulated
	   using common I2C messages */
	/* master_xfer should return the number of messages successfully
	   processed, or a negative value on error */
	int (*master_xfer)(struct i2c_adapter *adap, struct i2c_msg *msgs,
			   int num);
	int (*smbus_xfer) (struct i2c_adapter *adap, u16 addr,
			   unsigned short flags, char read_write,
			   u8 command, int size, union i2c_smbus_data *data);

	/* To determine what the adapter supports */
	u32 (*functionality) (struct i2c_adapter *);

#if IS_ENABLED(CONFIG_I2C_SLAVE)
	int (*reg_slave)(struct i2c_client *client);
	int (*unreg_slave)(struct i2c_client *client);
#endif
};
~~~



- `master_xfer`：函数指针，用于传输一系列I2C消息到I2C总线。返回成功处理的消息数量，或在出现错误时返回负值。
- `functionality`：功能函数指针，用于确定适配器支持的功能。





~~~c
static int rk3x_i2c_probe(struct platform_device *pdev)
{
	//获取设备树节点
	struct device_node *np = pdev->dev.of_node;
	const struct of_device_id *match;
	struct rk3x_i2c *i2c;
	struct resource *mem;
	int ret = 0;
	u32 value;
	int irq;
	unsigned long clk_rate;

	//申请空间
	i2c = devm_kzalloc(&pdev->dev, sizeof(struct rk3x_i2c), GFP_KERNEL);
	if (!i2c)
		return -ENOMEM;
	//遍历匹配表，找到与当前设备树节点（np）匹配的项，并返回匹配项的指针
	match = of_match_node(rk3x_i2c_match, np);
	//将匹配项的数据指针保存在 i2c->soc_data 字段中，以便后续在驱动程序中使用这些 SoC 相关的数据
	i2c->soc_data = match->data;

	/* use common interface to get I2C timing properties */
	i2c_parse_fw_timings(&pdev->dev, &i2c->t, true);

	// 将字符串 "rk3x-i2c" 复制到 i2c->adap.name 字段中，用于指定 I2C 适配器（I2C Adapter）的名称。
	strlcpy(i2c->adap.name, "rk3x-i2c", sizeof(i2c->adap.name));
	// 设置适配器所有者（owner），使用内核模块（THIS_MODULE）作为适配器的拥有者。
	i2c->adap.owner = THIS_MODULE;
	// 关联 I2C 适配器与特定的 I2C 算法（Algorithm）。这里，&rk3x_i2c_algorithm 是指向特定算法实现的指针。
	i2c->adap.algo = &rk3x_i2c_algorithm;
	// 设置 I2C 适配器的重试次数，指示在出现数据传输错误时重试的次数。
	i2c->adap.retries = 3;
	// 将设备树节点（np）关联到 I2C 适配器的设备节点（dev.of_node）字段中，以便在适配器操作中获取设备树信息。
	i2c->adap.dev.of_node = np;
	// 设置 I2C 适配器的算法数据字段（algo_data）为 i2c，即指向 I2C 控制器结构体的指针，以便在算法操作中访问控制器相关数据。
	i2c->adap.algo_data = i2c;
	// 设置 I2C 适配器的设备父节点（dev.parent）字段为平台设备结构体（pdev->dev）的指针，以关联适配器与相应的父设备。
	i2c->adap.dev.parent = &pdev->dev;
	// 将 I2C 控制器的设备指针（dev）设置为平台设备结构体（pdev->dev）的指针，以便在驱动程序中使用该指针进行设备操作。
	i2c->dev = &pdev->dev;
    
	spin_lock_init(&i2c->lock);
	init_waitqueue_head(&i2c->wait);

	i2c->i2c_restart_nb.notifier_call = rk3x_i2c_restart_notify;
	i2c->i2c_restart_nb.priority = 128;
	ret = register_pre_restart_handler(&i2c->i2c_restart_nb);
	if (ret) {
		dev_err(&pdev->dev, "failed to setup i2c restart handler.\n");
		return ret;
	}

	mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
	i2c->regs = devm_ioremap_resource(&pdev->dev, mem);
	if (IS_ERR(i2c->regs))
		return PTR_ERR(i2c->regs);

	/*
	 * Switch to new interface if the SoC also offers the old one.
	 * The control bit is located in the GRF register space.
	 */
	if (i2c->soc_data->grf_offset >= 0) {
		struct regmap *grf;

		grf = syscon_regmap_lookup_by_phandle(np, "rockchip,grf");
		if (!IS_ERR(grf)) {
			int bus_nr;

			/* Try to set the I2C adapter number from dt */
			bus_nr = of_alias_get_id(np, "i2c");
			if (bus_nr < 0) {
				dev_err(&pdev->dev, "rk3x-i2c needs i2cX alias");
				return -EINVAL;
			}

			if (i2c->soc_data == &rv1108_soc_data && bus_nr == 2)
				/* rv1108 i2c2 set grf offset-0x408, bit-10 */
				value = BIT(26) | BIT(10);
			else if (i2c->soc_data == &rv1126_soc_data &&
				 bus_nr == 2)
				/* rv1126 i2c2 set pmugrf offset-0x118, bit-4 */
				value = BIT(20) | BIT(4);
			else
				/* rk3xxx 27+i: write mask, 11+i: value */
				value = BIT(27 + bus_nr) | BIT(11 + bus_nr);

			ret = regmap_write(grf, i2c->soc_data->grf_offset,
					   value);
			if (ret != 0) {
				dev_err(i2c->dev, "Could not write to GRF: %d\n",
					ret);
				return ret;
			}
		}
	}

	/* IRQ setup */
	irq = platform_get_irq(pdev, 0);
	if (irq < 0) {
		dev_err(&pdev->dev, "cannot find rk3x IRQ\n");
		return irq;
	}

	ret = devm_request_irq(&pdev->dev, irq, rk3x_i2c_irq,
			       0, dev_name(&pdev->dev), i2c);
	if (ret < 0) {
		dev_err(&pdev->dev, "cannot request IRQ\n");
		return ret;
	}

	platform_set_drvdata(pdev, i2c);

	if (i2c->soc_data->calc_timings == rk3x_i2c_v0_calc_timings) {
		/* Only one clock to use for bus clock and peripheral clock */
		i2c->clk = devm_clk_get(&pdev->dev, NULL);
		i2c->pclk = i2c->clk;
	} else {
		i2c->clk = devm_clk_get(&pdev->dev, "i2c");
		i2c->pclk = devm_clk_get(&pdev->dev, "pclk");
	}

	if (IS_ERR(i2c->clk)) {
		ret = PTR_ERR(i2c->clk);
		if (ret != -EPROBE_DEFER)
			dev_err(&pdev->dev, "Can't get bus clk: %d\n", ret);
		return ret;
	}
	if (IS_ERR(i2c->pclk)) {
		ret = PTR_ERR(i2c->pclk);
		if (ret != -EPROBE_DEFER)
			dev_err(&pdev->dev, "Can't get periph clk: %d\n", ret);
		return ret;
	}

	ret = clk_prepare(i2c->clk);
	if (ret < 0) {
		dev_err(&pdev->dev, "Can't prepare bus clk: %d\n", ret);
		return ret;
	}
	ret = clk_prepare(i2c->pclk);
	if (ret < 0) {
		dev_err(&pdev->dev, "Can't prepare periph clock: %d\n", ret);
		goto err_clk;
	}

	i2c->clk_rate_nb.notifier_call = rk3x_i2c_clk_notifier_cb;
	ret = clk_notifier_register(i2c->clk, &i2c->clk_rate_nb);
	if (ret != 0) {
		dev_err(&pdev->dev, "Unable to register clock notifier\n");
		goto err_pclk;
	}

	clk_rate = clk_get_rate(i2c->clk);
	rk3x_i2c_adapt_div(i2c, clk_rate);

	ret = i2c_add_adapter(&i2c->adap);
	if (ret < 0)
		goto err_clk_notifier;

	return 0;

err_clk_notifier:
	clk_notifier_unregister(i2c->clk, &i2c->clk_rate_nb);
err_pclk:
	clk_unprepare(i2c->pclk);
err_clk:
	clk_unprepare(i2c->clk);
	return ret;
}

~~~

## crypto子系统

### 算法的选择

内核使用算法名(cra_name)来表示某一特定的算法，而使用算法驱动名(cra_driver_name)来表示算法的特定实现。不同的crypto_alg可以含有相同的cra_name，但其cra_driver_name是唯一的。用户可以使用cra_name或cra_driver_name来调用相关算法，为了在用户使用cra_name参数时选择最优的算法实现，内核对每种crypto_alg都定义了一个优先级。当给定cra_name还有多种不同实现时，则选择其中优先级最高的算法实现。例如以下链表中，包含了两个ecb(aes)的算法实现，其中一个优先级为100，另一个优先级为400，则若用户指定了该算法，则内核会选择优先级为400的算法。

~~~c
//内核默认的算法
static struct shash_alg sha256_algs[2] = { {
	.digestsize	=	SHA256_DIGEST_SIZE,
	.init		=	crypto_sha256_init,
	.update		=	crypto_sha256_update,
	.final		=	crypto_sha256_final,
	.finup		=	crypto_sha256_finup,
	.descsize	=	sizeof(struct sha256_state),
	.base		=	{
		.cra_name	=	"sha256",
		.cra_driver_name=	"sha256-generic",
		.cra_priority	=	100,
		.cra_blocksize	=	SHA256_BLOCK_SIZE,
		.cra_module	=	THIS_MODULE,
	}
}, {
	.digestsize	=	SHA224_DIGEST_SIZE,
	.init		=	crypto_sha224_init,
	.update		=	crypto_sha256_update,
	.final		=	crypto_sha256_final,
	.finup		=	crypto_sha256_finup,
	.descsize	=	sizeof(struct sha256_state),
	.base		=	{
		.cra_name	=	"sha224",
		.cra_driver_name=	"sha224-generic",
		.cra_priority	=	100,
		.cra_blocksize	=	SHA224_BLOCK_SIZE,
		.cra_module	=	THIS_MODULE,
	}
} };

//硬件加速算法
struct shash_alg infinity_shash_sha256_alg = {.digestsize = SHA256_DIGEST_SIZE,
                                              .init       = infinity_sha256_init,
                                              .update     = infinity_sha256_update,
                                              .final      = infinity_sha256_final,
                                              .export     = infinity_sha256_export,
                                              .import     = infinity_sha256_import,
                                              .descsize   = sizeof(struct sha256_state),
                                              .statesize  = sizeof(struct sha256_state),
                                              .base       = {
                                                  .cra_name        = "sha256",
                                                  .cra_driver_name = "sha256-infinity",
                                                  .cra_priority    = 400,
                                                  .cra_flags       = CRYPTO_ALG_TYPE_SHASH | CRYPTO_ALG_NEED_FALLBACK,
                                                  .cra_blocksize   = SHA256_BLOCK_SIZE,
                                                  .cra_module      = THIS_MODULE,
                                                  .cra_ctxsize     = sizeof(struct infinity_sha256_ctx),
                                              }};
~~~

### 源码

* 内核加密算法源码：

crypto\aes_generic.c

crypto\sha256_generic.c

* 硬件加密算法

drivers\sstar\crypto\mdrv_aes.c

drivers\sstar\crypto\mdrv_sha.c

### 参考文献

* [linux加解密框架（一）基础介绍 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/548277837)
* [linux加解密框架（二）算法注册 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/548893037)
* [linux加解密框架（三）算法模板 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/549125733)
* [linux加解密框架（四）加解密流程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/550038832)

* [linux加解密框架（五）用户接口实现 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/550176751)
