# 调试工具

## JLINK

使用eclipse等IDE时可能会需要使用JLink进行程序烧录，如果使用图形界面的话需要在多个界面之间进行切换，不是很方便，这里可以通过Tools之类的IDE工具通过命令行直接调用JLink从而实现程序下载，具体实现如下。

JLink有一个很实用的命令行参数是-CommandFile <\*.jlink>，通过这个参数可以向JLink程序传递我们需要执行的命令，这个*.jlink文件的内容其实就是JLink内部命令的有序排列，下面的示例便是我在烧录STM32F030K6的程序时使用到的jlink文件。

```shell
Device STM32F030K6
si SWD
speed 4000
loadfile .\bin\Release\stm32f030.hex
r
g
exit
```
Device 命令指定目标芯片
si 命令选择调试器使用的总线，如SWD、JTAG等
speed 指定传输速率
loadfile 命令加载目标文件至单片机，也可以使用loadbin
r g exit 依次是复位、运行然后退出JLink.exe程序

将上面的这几句保存成文件load.jlink，放到工程目录中去，然后使用IDE提供的外部调用工具便能直接将程序烧录至单片机了。

## DAPLINK

CMSIS DAP是ARM推出的开源方案的调试器，目前该项目在Github上已更名为DAPLink，ARM官方推荐的GDB Server是pyocd，也可以使用openocd来作为调试服务器。

之前从恩智浦技术社区论坛申请到了万利的开发板LPC54114，该板板载CMSIS DAP调试器，可以很方便的进行调试。最初我打算使用openocd来作为GDB Server，但接下来我发现目前版本的openocd并没有提供LPC54114微控制器的flash烧录算法支持，然后只能使用pyocd了，然而Release版本(2018年3月左右，目前的版本已提供了LPC54114支持)的pyocd同样没有提供支持,但是我在github仓库中发现已经有LPC54114的支持，然后按照提示从源码进行了安装，然后就可以使用了。

### 更新固件

linux系统下更新固件无法使用拖拽式更新，需要使用如下的命令

```shell
dd if={new_firmware.bin} of={firmware.bin} conv=notrunc
```

### pyocd

目前很少会使用pyocd作为gdbserver来进行单步调试，主要用它来下载程序，命令如下，参数都比较一目了然了

```shell
pyocd flash -t lpc54114 xxx.bin
```

查看pyocd支持的target列表可以使用下面的命令

```shell
pyocd list --target
```

其他功能的使用可以参考帮助信息以及pyocd的[支持文档](https://github.com/mbedmicro/pyOCD/blob/master/docs/target_support.md)。

### openocd

至于openocd，虽然没有使用，但在这里也做些记录，方便以后使用。下面文档来源于[CMSIS-DAP和openOCD那些事](https://blog.csdn.net/m454078356/article/details/78986205),主要步骤如下(以stm32为例)

1. 新建一个配置文件ocd-stm32.cfg，使用swd接口

```shell
interface cmsis-dap
transport select swd
source [find target/stm32f1x.cfg] 
```

2. 连接CMSIS-DAP和stm32f103，加载脚本

```shell
openocd -f ./ocd-stm32.cfg
```

3. 这样显示就是连接成功了

```shell
Open On-Chip Debugger 0.9.0 (2015-05-19-12:09)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : only one transport option; autoselect 'swd'
Warn : Transport "swd" was already selected
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
none separate
cortex_m reset_config sysresetreq
Info : CMSIS-DAP: SWD  Supported
Info : CMSIS-DAP: Interface Initialised (SWD)
Info : CMSIS-DAP: FW Version = 1.0
Info : SWCLK/TCK = 1 SWDIO/TMS = 1 TDI = 0 TDO = 0 nTRST = 0 nRESET = 1
Info : CMSIS-DAP: Interface ready
Info : clock speed 1000 kHz
Info : SWD IDCODE 0x1ba01477
Info : stm32f1x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

openOCD默认端口telnet 4444，gdb server 3333，然后telnet连接到 127.0.0.1:4444，就可以为所欲为了。

```shell
下载一个hex文件到flash
flash write_image erase ./F103.hex
```
