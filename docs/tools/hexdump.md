# Hexdump

> hexdump, hd — ASCII, decimal, hexadecimal, octal dump

> The hexdump utility is a filter which displays the specified files, or the standard input, if no files are specified, in a user specified format.

hexdump除了直接查看二进制文件以外还可以通过其格式化输出功能实现一些比较方便的功能，例如将二进制文件转换为16进制文本文件，使用`-e`参数来指定格式，man手册中提供的示例如下：

```shell
# hex bytes
% echo hello | hexdump -v -e '/1 "%02X "' ; echo
68 65 6C 6C 6F 0A

# same, with ASCII section
% echo hello | hexdump -e '8/1 "%02X ""\t"" "' -e '8/1 "%c""\n"'
68 65 6C 6C 6F 0A        hello
```

两个示例的功能分别如下：

* 每次读取一个字节，并使用`%02X `格式来格式化输出
* 第一个`-e`含义为每次读取一个字节，以8字节为一组使用`"%02X ""\t"" "`格式来格式化输出，第二个同理

所以，将第二个示例的输入修改为大于8个字符时，其输出如下：

```shell
$ echo helloooo | hexdump -e '8/1 "%02X ""\t"" "' -e '8/1 "%c""\n"'
68 65 6C 6C 6F 6F 6F 6F  helloooo
0A                       
```

可以发现，合理的使用格式化输出可以实现各种形式的二进制和字符转换。

例如在调试verilog程序中需要生成一些汇编指令码注入rom模块，通常我们使用汇编编写需要的程序并编译生成二进制文件，然后将其转换为16进制ascii文件并通过testbench进行读取，这便需要一个程序来完成转换工作。

假设有一个需要转换的bin文件，其内容如下：

```shell
$ hexdump -C inst_rom.bin 
00000000  00 11 01 34 20 00 21 34  00 44 21 34 44 00 21 34  |...4 .!4.D!4D.!4|
00000010  02 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 20 01 01 00 00 03  |.......... .....|
00000030  00 00 00 00 00 00 00 00  01 00 00 00 00 00 00 00  |................|
00000040
```

需要将其以4字节为单位转换为ascii文件，这时候hexdump便可以出马了，使用如下命令:

```shell
$ hexdump -v -e '/4 "%08x\n"' inst_rom.bin
34011100
34210020
34214400
34210044
00000002
00000000
00000000
00000000
00000000
00000000
01200000
03000001
00000000
00000000
00000001
00000000
```

bingo!

已经非常完美的转换出来了，只要将输出重定向到文件中便完美的解决问题了。
