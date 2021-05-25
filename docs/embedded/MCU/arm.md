# ARM

## 1. _exit问题

链接时报错，信息如下

```shell
gcc-arm-none-eabi-8-2019-q3-update/bin/../lib/gcc/arm-none-eabi/8.3.1/../../../../arm-none-eabi/lib/thumb/v7e-m+fp/hard/libc.a(lib_a-exit.o): in function `exit':
exit.c:(.text.exit+0x16): undefined reference to `_exit'
collect2: error: ld returned 1 exit status
```

造成上述问题的原因是链接时未指定`--specs=nosys.specs`，增加该参数即可解决此问题.

此外，如果需要semihosting，链接如下：

```shell
$ arm-none-eabi-gcc --specs=rdimon.specs $(OTHER_LINK_OPTIONS)
```

如果使用retarget（重定向），链接如下：

```shell
$ arm-none-eabi-gcc --specs=nosys.specs $(OTHER_LINK_OPTIONS)
```
