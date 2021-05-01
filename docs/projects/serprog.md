# serprog

serprog是[flashrom](http://flashrom.org/)支持的众多编程器中的一种，可以通过串口操作单片机来给spi flash编程，在flashrom的[wiki](https://flashrom.org/Serprog)中介绍了多种平台的实现。

我最开始从[Linux 下离线烧写 SPI 闪存](http://blog.dword1511.info/?p=4107)这篇博客中了解到这种编程器，对此比较感兴趣，于是仔细阅读了一下原博主的文章和代码仓库，正好手头有一款很久之前免费拿到的lpc54114-lite开发板，于是便在该开发板上进行了简单实现。

该板卡使用了nxp lpc54114双核mcu，具有USB全速设备，正好可以满足serprog的要求。

代码编写完成后进行了简单的测试，结果如下：

```shell
# 读flash, W25X80, 1MiB
$ time sudo flashrom -p serprog:dev=/dev/ttyACM0:4000000 -c W25X80 -r /tmp/test.bin
flashrom  on Linux 4.19.0-16-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
serprog: Programmer name is "lpc-serprog"
Found Winbond flash chip "W25X80" (1024 kB, SPI) on serprog.
Reading flash... done.

real    0m3.106s
user    0m1.163s
sys     0m0.055s

# 擦除flash, W25X80, 1MiB
$ time sudo flashrom -p serprog:dev=/dev/ttyACM0:4000000 -c W25X80 -E
flashrom  on Linux 4.19.0-16-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
serprog: Programmer name is "lpc-serprog"
Found Winbond flash chip "W25X80" (1024 kB, SPI) on serprog.
Erasing and writing flash chip... Erase/write done.

real    0m25.594s
user    0m23.017s
sys     0m0.065s

# 写flash, W25X80, 1MiB
$ time sudo flashrom -p serprog:dev=/dev/ttyACM0:4000000 -c W25X80 -w /tmp/test.bin 
flashrom  on Linux 4.19.0-16-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
serprog: Programmer name is "lpc-serprog"
Found Winbond flash chip "W25X80" (1024 kB, SPI) on serprog.
Reading old flash chip contents... done.
Erasing and writing flash chip... Erase/write done.
Verifying flash... VERIFIED.

real    0m17.041s
user    0m2.757s
sys     0m0.717s
```

从读写数据来看效果还是不错了，虽然相较于原博主的实现读写速度略有不急，不过日常拿来使用也算够用了。

目前相关源码托管于[github](https://github.com/liweihao-cn/serprog-lpc54114-lite)以及[gitee](https://gitee.com/liweihao_cn/serprog-lpc54114-lite)。
