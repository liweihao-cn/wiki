# Linux SOC级内核移植实践

<a id="porting_linux_kernel_to_new_soc"></a>

> 一直以来都想学习如何将Linux移植到一个架构已经被支持的新SOC上，现在终于有时间来进行一些实践练习了，这个笔记记录了整个实践过程以及在移植过程中踩到的一些坑。我只有mini2440和licheepi zero两个平台的开发板，想比较而言，s3c2440的内部结构比较简单，所以这里将s3c2440假设成一个新的SOC，暂且命名为foo吧，移植过程采用了设备树的机制，下面就是正文了。

## 1. 移植前需要了解的一些信息

Linux的移植有三种级别：

* 移植到一个新的架构上
* 移植到一个架构已经被支持的SOC上
* 移植到一个新的板卡上

架构级别的移植像我们这样的普通开发者很难会接触到，板卡级别的移植网上已经有相当多的文档了，而SOC级别移植的工作往往由芯片原厂来进行，这次的移植过程就大量参考了内核中不同厂商的代码。

那么，要让linux运行在一个新的SOC上需要做哪些工作呢，我在网上找到了两篇文档：

* [arm-soc-checklist.pdf](../../assets/pdf/arm-soc-checklist.pdf)
* [porting-linux-to-a-new-soc.pdf](../../assets/pdf/porting-linux-to-a-new-soc.pdf)

从文档中我们可以了解到，由于SOC采用的架构已经被linux支持，这意味着内存管理部分已经是可用的，要让linux运行起来，我们只需要实现**中断控制器驱动**、**定时器驱动**、**串口驱动**这三部分。当然实现这些驱动只是最基础的部分，要让各个外设正常运行还需要大量的驱动开发工作。

## 2. 搭建编译框架

> 编译框架部分的搭建参考了mach-mxs平台的代码

1. 在arch/arm/下新建mach-foo目录，在该目录下新建Kconfig、Makfile、mach-foo.c文件，修改后的目录树如下：

```shell
arch/arm/mach-foo
├── Kconfig
├── mach-foo.c
└── Makefile
```

2. 修改mach-foo中的三个文件，分别添加如下内容：

```c
/* mach-foo.c */
#include <linux/io.h>
#include <linux/compiler.h>

#include <asm/mach/arch.h>

static void __init foo_machine_init(void)
{
}

static const __initconst char *const foo_dt_compat[] = {
	"none,foo",
	NULL,
};

DT_MACHINE_START(foo, "mach-foo(DT)")
	.init_machine	= foo_machine_init,
	.dt_compat	= foo_dt_compat,
MACHINE_END
```

```shell
#Makefile
obj-$(CONFIG_ARCH_FOO) += mach-foo.o
```

```shell
#Kconfig
config ARCH_FOO
	bool "foo(actually s3c2440) SOC support"
	depends on ARCH_MULTI_V4T
	select CPU_ARM920T
	select GPIOLIB
	select PINCTRL
	select SOC_BUS
	help
	  Support for my test project.
```

这里的修改参考了mach-mxs中的代码，**CPU_ARM920T**这个宏用来定义CPU的类型，内核一开始检测CPU是否匹配也是通过这个宏来选择CPU型号的，具体信息可以从[这里](arch-arm.md#__lookup_processor_type)查看。Kconfig中其余的选项都是直接从mach-foo目录下抄来的，暂时可以不用理会，不过这个**SOC_BUS**后面可能会用到，也是一个比较有趣的特性，可以从[这里](bus.md#CONFIG_SOC_BUS)查看细节。

3. 修改上级目录(arch/arm)中的Kconfig、Makefile，主要修改如下：

```shell
#arch/arm/Kconfig
source "arch/arm/mach-foo/Kconfig"
```

```shell
#arch/arm/Makefile
machine-$(CONFIG_ARCH_FOO)		+= foo
```

观察这两个文件里面的原始内容可以发现是按照英文字母顺序来排序的，所以我们添加这两处的时候也要遵循这个规则，这里的**CONFIG_ARCH_FOO**是新添加的SOC的宏，对应第2步中Kconfig中的内容。

4. 在arch/arm/boot/dts中新建两个文件foo.dtsi和foo-evk.dts，并在里面填充适当的内容，其中的cpus节点和memory节点要根据具体情况来填写

```c
/* foo.dtsi */
#include "skeleton.dtsi"

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/interrupt-controller/irq.h>

/ {
	model = "foo SOC";
	compatible = "none,foo";

	cpus {
		#address-cells = <0>;
		#size-cells = <0>;

		cpu {
			compatible = "arm,arm920t";
		};
	};
};
```

```c
/* foo-evk.dts */
/dts-v1/;
#include "foo.dtsi"

/ {
	model = "foo board";
	compatible = "none,foo";

	memory {
		reg = <0x30000000 0x04000000>;
	};
};
```

修改arch/arm/boot/dts/Makefile,增加设备树的编译选项

```shell
#arch/arm/boot/dts/Makefile
dtb-$(CONFIG_ARCH_FOO) += \
	foo-evk.dtb
```

5. 拷贝mxs_defconfig到foo_defconfig作为我们的默认配置模板，然后执行make menuconfig选择我们上面添加的配置，并适当的删减一些目前用不到的驱动，减少编译时间。

最后执行编译，如果zImage和dtb都能正确生成的话就可以把当前的.config保存成foo_defconfig了，具体的保存步骤可以从[这里](config.md#savedefconfig)查看。


上述修改可以参考[这个提交记录](https://github.com/liweihao-cn/linux-4.19.108/commit/c5fc0821e01cb850855d87f8ce30bc93808110b2)。

## 3. 实现earlycon驱动

> 此部分的修改参考了[X-012-KERNEL-serial early console的移植](http://www.wowotech.net/x_project/kernel_earlycon_porting.html)、[Linux earlycon made life easy](https://mystictot.wordpress.com/2018/11/04/linux-earlycon-made-life-easy/)、kernel-parameters.txt以及内核中其他SOC的实现。

在正式的串口驱动可用之前命令行参数指定的终端是无法输出任何信息的，为此Linux提供了两种机制来进行早期的log输出，分别是early_printk和earlycon，从网上的种种说法来看，earlycon是一种更好、更优雅的方式，所以这里采用earlycon作为我们早期log打印的方式。

在u-boot中串口已经初始化过了，所以这里串口可以直接使用，参考[X-012-KERNEL-serial early console的移植](http://www.wowotech.net/x_project/kernel_earlycon_porting.html)，在drivers/tty/serial/下新建foo-serial.c，并填充下面的内容，这里不同的地方是借助earlycon的框架来映射的内存，没有手动进行内存映射。

```c
/* drivers/tty/serial/foo-serial.c */
#include <linux/kernel.h>
#include <linux/console.h>
#include <linux/init.h>
#include <linux/serial_core.h>

#include <asm/io.h>
#include <asm/early_ioremap.h>

#define UART_UTRSTAT	(0x10)
#define UART_UTXH	(0x20)

static void foo_serial_putc(struct uart_port *port, int ch)
{
	while (!(readl(port->membase + UART_UTRSTAT) & 0x2));
	writeb(ch, port->membase + UART_UTXH);
}

static void earlycon_foo_write(struct console *con, const char *s, unsigned n)
{
	struct earlycon_device *dev = con->data;

	uart_console_write(&dev->port, s, n, foo_serial_putc);
}

int __init earlycon_foo_setup(struct earlycon_device *dev, const char *opt)
{
	if (!dev->port.membase)
		return -ENODEV;

	dev->con->write = earlycon_foo_write;
	return 0;
}
EARLYCON_DECLARE(foo_serial, earlycon_foo_setup);
```

这里使用设备树来传递earlycon需要的参数，所以在foo-evk.dts中添加下面的内容，注意这里添加了用于映射的地址，earlycon默认会映射64字节大小的空间

```c
	chosen {
		bootargs = "earlycon=foo_serial,0x50000000 console=ttyS0";
	};
```

这里之所以添加了一个额外的console参数是因为如果没有这个参数的话内核会很快的切换到默认console(tty0)上，之后所有的启动日志便看不到了，但我们并不想这样，所以在这里指定一个非默认的console设备。

参考其余厂商驱动对Makefile和Kconfig进行修改，并通过menuconfig配置支持printk的输出,需要使能下面这个选项

```c
General setup  --->
	[*] Configure standard kernel features (expert users)  --->
```

一切顺利的话这个时候就可以通过串口看到kernel的启动过程了，此部分修改可以参考[这个提交记录](https://github.com/liweihao-cn/linux-4.19.108/commit/28010cf66b2ce602f45e7e2f7080fadee65ce9a3)。

## 4. 实现irqchip驱动

> 很多的驱动都依赖于中断，比如定时器，而定时器驱动是内核必须的，所以中断的驱动是必须的，这里irqchip驱动的实现参考了irq-lpc32xx.c、irq-mxs.c、三星的驱动代码以及蜂窝科技论坛的文章。

受益于内核框架的巧妙，大部分时候我们不需要对具体细节进行了解也可以开发出正常工作的驱动代码。内核驱动开发很多时候就是将一个设备进行抽象，然后填充到内核提供的框架中去。

[中断子系统-蜂窝科技](http://www.wowotech.net/sort/irq_subsystem)系列文章对linux中断子系统进行了很详细的介绍，从这些文章中我们可以了解到很多细节，但是本次移植主要是参考了内核中一些厂商的代码。

s3c2440的中断控制器将中断源分为了main和sub两种，要想自己写一份驱动，先读一下三星提供的驱动再上手也不迟。厂商驱动总是会一份驱动兼容多款SOC，有时会为我们阅读代码带来一些障碍。这里对比阅读了上面提到的几份代码，又考虑到s3c3440中断控制器的设计，最后决定参照irq-lpc32xx.c中的实现方式来编写foo的irqchip驱动。

由于移植过程比较复杂，且并不通用，但是整体思想是一致的，所以这里只是简要介绍一下移植的流程。

首先将抽象好的中断控制器在设备树中进行描述，下面是我自己对2440中断控制器的描述

```c
	mic: mic@4a000000 {
		compatible = "foo,foo-mic";
		reg = <0x4a000000 0x20>;
		interrupt-controller;
		#interrupt-cells = <2>;
	};

	sic: sic@4a000000 {
		compatible = "foo,foo-sic";
		reg = <0x4a000000 0x20>;
		interrupt-controller;
		#interrupt-cells = <2>;

		interrupt-parent = <&mic>;
		interrupts = <6 IRQ_TYPE_LEVEL_LOW>,
			     <9 IRQ_TYPE_LEVEL_LOW>,
			     <15 IRQ_TYPE_LEVEL_LOW>,
			     <16 IRQ_TYPE_LEVEL_LOW>,
			     <23 IRQ_TYPE_LEVEL_LOW>,
			     <28 IRQ_TYPE_LEVEL_LOW>,
			     <31 IRQ_TYPE_LEVEL_LOW>;
	};
```

设备树抽象好之后就可以开始编写驱动文件了，在drivers/irqchip中新建irq-foo.c文件，并修改Makefile。然后在文件中使用**IRQCHIP_DECLARE**宏定义一个初始化函数，在这个初始化函数里主要完成的工作有

* 寄存器地址的映射
* irq_domain的添加及该domain对应的ops(主要是map和xlate)的实现
* 设置irq_chip的mask/unmask/ack等回调，这个是和具体中断控制器相关的
* 设置irq的handle
* 设置chained类型的handle和data(这一步不同的中断控制器可能不一样，有的不需要)

实现驱动的过程中可能还会需要维护一些驱动私有的数据结构以便进行一些操作，在这里就维护了一份mic对应的sic的sub_bits数据，以便在mask/unmask的时候做一些判断，这部分参考了三星驱动中的做法。

irqchip的驱动移植到这里就结束了，经过测试mic是可以正常的工作的，而sic还没有测试，后续编写串口驱动的时候便会用到sic了，到时候再看sic能否正常工作。整个移植过程中参考了三星提供的驱动以及一些其他厂商的驱动，多读一读这些代码便可以熟悉驱动编写的流程了，上述改动内容都在[这个提交记录](https://github.com/liweihao-cn/linux-4.19.108/commit/bfd8a4be09689ab85556d9e9f671cc0885ae986d)中。

## 5. 实现clk驱动

> 这部分驱动的编写参考了三星提供的驱动和全志的simple-gates驱动。

clk驱动属于电源管理子系统的一部分,主要功能是实现soc内部各模块和外设的时钟管理,编写这一部分驱动的时候需要对CCF(Common Clock Framework)有些简单的了解,这部分内容可以参考[电源管理子系统-蜂窝科技](http://www.wowotech.net/sort/pm_subsystem)。

由于对clock各个部分进行分类比较复杂，况且这里也不需要特别复杂的时钟管理功能，而且在u-boot中已经将fclk、hclk和pclk初始化好了，所以这里直接简单粗暴将fclk、hclk和pclk注册成fixed rate clock，内核fixed rate相关的部分会自动为我们注册对应的clock，设备树如下所示

```c
	clocks {
		compatible = "simple-bus";
		#address-cells = <1>;
		#size-cells = <1>;

		xti: xti {
			compatible = "fixed-clock";
			clock-frequency = <12000000>;
			clock-output-names = "xti";
			#clock-cells = <0>;
		};

		fclk: fclk {
			compatible = "fixed-clock";
			clock-frequency = <400000000>;
			clock-output-names = "fclk";
			#clock-cells = <0>;
		};

		hclk: hclk {
			compatible = "fixed-clock";
			clock-frequency = <100000000>;
			clock-output-names = "hclk";
			#clock-cells = <0>;
		};

		pclk: pclk {
			compatible = "fixed-clock";
			clock-frequency = <50000000>;
			clock-output-names = "pclk";
			#clock-cells = <0>;
		};
	};
```

然后参考全志的simple-gates驱动编写我们的gates设备树及驱动，设备树如下所示

```c
	clk: clock-controller@0x4c000000 {
		compatible = "foo,foo-clock";
		reg = <0x4c000000 0x40>;
		#clock-cells = <1>;

		clock-indices = <4>, <5>, <6>, <7>, 
				<8>, <9>, <10>,<11>,
				<12>, <13>, <14>, <15>,
				<16>, <17>, <18>, <19>,
				<20>;
		clock-parent-names = "hclk","hclk","hclk","pclk",
				     "pclk","pclk","pclk","pclk",
				     "pclk","pclk","pclk","pclk",
				     "pclk","pclk","pclk","hclk",
				     "pclk";
		clock-gates-names = "nand", "lcdc", "usb-host", "usb-device",
				    "timer", "sdi", "uart0", "uart1",
				    "uart2", "gpio", "rtc", "adc",
				    "iic", "iis", "spi", "camera",
				    "ac97";
	};
```

具体的代码可以查看[这个提交记录](https://github.com/liweihao-cn/linux-4.19.108/commit/aafe268f9b23700f58fb858e8fdca952e7524040)。

这里有一点需要注意，如果某个外设的时钟在注册gate之前是开着的，那么一定要在注册完gate之后通过内核接口使能一下该时钟，代码里的foo_critical_clocks就是起这个作用的，否则可能会在initcall阶段调用clk_disable_unused时导致阻塞，我猜测这里阻塞可能是因为状态不一致导致的，可以通过initcall_debug来调试initcall的执行，可以从[这里](parameters.md#initcall_debug)了解细节。

## 6. 实现timekeeping驱动

最开始看三星提供的pwm timer驱动时有点懵，三星的驱动应该是使用同一个定时器既作为clocksource又作为clcokevent，从他们的功能分类可以发现这俩按道理应该使用不同定时器，[这篇博客](https://blog.csdn.net/pwl999/article/details/78203045)对time部分做了比较详细的介绍，里面也提到了这两个功能应该是由两个硬件部分来实现的。所以这里的移植决定采用两个定时器来实现，一个提供中断，而另一个不提供中断，通过设备树中的一些字段来进行区分，不过这样做似乎不是很优雅，不过还是先实现功能再说吧。

最开始实现驱动的时候我想提高一些精度，于是将定时器的时钟分频到了5M，但实际运行的时候发现启动时候的时间戳出现了很严重的回绕现象，这部分时间戳的打印应该是由sched_clock部分通过我们用sched_clock_register注册的读函数来实现的，但是s3c2440的定时器只有16位，可能是提高定时器频率后溢出太快导致内核还没来得及查询导致的，所以最后还是将定时器时钟频率调节到了1M。

这部分的修改可以查看[这个提交](https://github.com/liweihao-cn/linux-4.19.108/commit/301fb62c3c1852686be590da42d69f9575175ecd)。

## 7. 实现uart驱动
