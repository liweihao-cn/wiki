# ARM

主要记录一些内核中arm架构相关的内容

## 1. 处理器类型检查

<a id="__lookup_processor_type"></a>

**__lookup_processor_type** 检查处理器是否支持，其中用到了 **__lookup_processor_type_data** 这个数据，注释说明了它的数据类型定义在 **arch/arm/include/asm/procinfo.h** 中，进入这个文件中，从注释来看，这部分数据是在 **arch/arm/mm/proc-*.S** 中定义的，而这些文件的编译与否则依赖于我们选择的CPU类型，Makefile如下

```makefile
obj-$(CONFIG_CPU_ARM7TDMI)	+= proc-arm7tdmi.o
obj-$(CONFIG_CPU_ARM720T)	+= proc-arm720.o
obj-$(CONFIG_CPU_ARM740T)	+= proc-arm740.o
obj-$(CONFIG_CPU_ARM9TDMI)	+= proc-arm9tdmi.o
obj-$(CONFIG_CPU_ARM920T)	+= proc-arm920.o
obj-$(CONFIG_CPU_ARM922T)	+= proc-arm922.o
obj-$(CONFIG_CPU_ARM925T)	+= proc-arm925.o
obj-$(CONFIG_CPU_ARM926T)	+= proc-arm926.o
obj-$(CONFIG_CPU_ARM940T)	+= proc-arm940.o
obj-$(CONFIG_CPU_ARM946E)	+= proc-arm946.o
obj-$(CONFIG_CPU_FA526)		+= proc-fa526.o
obj-$(CONFIG_CPU_ARM1020)	+= proc-arm1020.o
obj-$(CONFIG_CPU_ARM1020E)	+= proc-arm1020e.o
obj-$(CONFIG_CPU_ARM1022)	+= proc-arm1022.o
obj-$(CONFIG_CPU_ARM1026)	+= proc-arm1026.o
obj-$(CONFIG_CPU_SA110)		+= proc-sa110.o
obj-$(CONFIG_CPU_SA1100)	+= proc-sa1100.o
obj-$(CONFIG_CPU_XSCALE)	+= proc-xscale.o
obj-$(CONFIG_CPU_XSC3)		+= proc-xsc3.o
obj-$(CONFIG_CPU_MOHAWK)	+= proc-mohawk.o
obj-$(CONFIG_CPU_FEROCEON)	+= proc-feroceon.o
obj-$(CONFIG_CPU_V6)		+= proc-v6.o
obj-$(CONFIG_CPU_V6K)		+= proc-v6.o
obj-$(CONFIG_CPU_V7)		+= proc-v7.o proc-v7-bugs.o
obj-$(CONFIG_CPU_V7M)		+= proc-v7m.o
```

## 2. TEXT_OFFSET

<a id="TEXT_OFFSET"></a>

**TEXT_OFFSET** 定义了内核镜像相对于基地址的偏移，这里的基地址，既包括**物理地址(PHYS_OFFSET)**，也包括**虚拟地址(PAGE_OFFSET)**，**TEXT_OFFSET**定义在**arch/arm/Makefile**中

```makefile
TEXT_OFFSET := $(textofs-y)
...
textofs-y	:= 0x00008000
```
绝大多数的SOC都使用这个默认配置，这也是为什么加载内核镜像到内存时的偏移为8000的原因。
