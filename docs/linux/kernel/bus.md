# 总线

<a id="bus"></a>

## 1. SOC_BUS

<a id="CONFIG_SOC_BUS"></a>

阅读代码的时候发现了**CONFIG_SOC_BUS**这个配置项，开启后会添加**drivers/base/soc.c**文件，这部分程序在运行时会注册一个名为**soc**的总线，并向外导出了**soc_device_register**函数。在soc级别的移植过程中可以通过这个函数向soc总线上注册一个soc设备，注册时传入一个**struct soc_device_attribute**类型的结构体变量指针，这个结构体的声明如下

```c
struct soc_device_attribute {
	const char *machine;
	const char *family;
	const char *revision;
	const char *soc_id;
};
```

最终会在sysfs中生成SOC相关的信息的文件。
