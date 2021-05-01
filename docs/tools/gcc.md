# GCC

## 指定字符编码

`-finput-charset`指定源文件（保存文件时选择）的编码方式（若不指定，编译器默认是UTF-8）
`-fexec-charset`指定可执行程序中的字符以什么编码方式来表示，默认是UTF-8

第二个参数是比较有用的，当使用gcc开发单片机显示相关程序时若使用的字库是GBK或GB2312编码时需要使用这个参数来指定程序中字符串的编码。

## 去除未用到的函数

在编译时添加`-ffunction-sections -fdata-sections`参数，在链接时添加`-Wl,-gc-sections`参数即可自动去除未使用的函数，减小代码二进制体积。

## 重定向

嵌入式开发中可以将`printf`函数进行重定向以便可以使用`printf`函数进行格式化输出。

增加`_write`函数的自定义实现，在自定义实现的函数中将buf中的数据通过串口进行发送即可实现重定向，该函数原型如下:

```c
int _write (int fd, const void *buf, size_t count);
```

需要注意的是，`\n`需要做特殊处理，当检测到传入的字符串最后一个字符为`\n`时，需要额外增加一个`\r`字符，这样便符合Linux系统下的使用情况了，以下是一个简单实例：

```c
int _write (int fd, const void *buf, size_t count)
{
    uint8_t *p = (uint8_t *)buf;
    (void) fd;
    USART_WriteBlocking(USART0, p, count);
    if (*(p+count-1) == '\n')
        USART_WriteByte(USART0, '\r');
    return count;
}
```

除此之外，还需在链接阶段增加`--specs=nano.specs --specs=nosys.specs`参数以使能重定向(retarget)并使用nano版本的libc，减小flash及ram的消耗。

## 参数传递

可以通过`-Wl,option`参数来传递参数给链接器；同理，通过`-Wa,option`可以传递参数给汇编器。

## 生成依赖关系

在使用Makefile组织项目的时候，可以在CFLAGS中增加`-MMD -MP`参数，这样GCC便会生成Makefile风格的依赖文件，借助这些依赖文件便可以实现修改头文件后自动重新编译所有相关的源码。
