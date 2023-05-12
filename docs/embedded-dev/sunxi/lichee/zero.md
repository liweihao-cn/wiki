# 荔枝派ZERO

荔枝派ZERO是sipeed推出的一款基于全志V3S SOC的迷你开发板，详细仓库可以从[官方文档](http://zero.lichee.pro/index.html)进行查看。

## Boot Select

V3S内置32KB BROM，启动过程中会依次检测BSP Pin状态、SD卡镜像、SPI flash镜像。如果BSP Pin为0，会直接进入FEL模式，如果在SD卡或SPI flash中检测到有效镜像，则会从相应介质中进行启动。如果BSP Pin不为0且未在SD卡和SPI flash中检测到有效镜像，那么最终也会进入FEL模式。

> 在全志V3S用户手册中并未找到BSP Pin的具体说明，网上资料显示，BSP Pin是SPI MISO引脚，经过测试，确实有效。

## FEL MODE

当SOC进入FEL模式后，可以通过USB连接PC并使用[sunxi-tools](https://github.com/linux-sunxi/sunxi-tools)进行设备检测和镜像烧录等操作。

进入FEL模式的途径除上述Boot Select外还有以下途径：

1. 使用一个特殊的SD卡镜像，镜像制作方法如下：

```shell
wget https://github.com/linux-sunxi/sunxi-tools/raw/master/bin/fel-sdboot.sunxi
dd if=fel-sdboot.sunxi of=/dev/sdX bs=1024 seek=8
```

2. 如果U-Boot可用的话，可以使用如下命令进入FEL模式：

```
go 0xffff0020
```

以上信息均来自[https://linux-sunxi.org/FEL](https://linux-sunxi.org/FEL)。

## 参考链接

* [https://linux-sunxi.org](https://linux-sunxi.org)
