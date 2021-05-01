# Linux Tips

Linux系统中的一些小工具、软件配置以及其他一些小玩意儿

* linux中最简单生成一个uuid的方法

```shell
$ cat /proc/sys/kernel/random
```

* 将sd卡/U盘格式化为无分区的样子，其实是覆盖了MBR

```shell
$ sudo mkfs.vfat -F 32 -I /d
```

* 检测网线插拔

```shell
$ /sys/class/net/eth0/carrier
```

* 32位运行库

```shell
$ apt install lib32z1
```

* 字符转艺术字

```shell
$ figlet
```

* linux查看运行时长

```shell
$ cat /proc/uptime
```

* 反向查询命令行历史记录，超实用

```shell
$ Ctrl+r
```

* linux串口工具

```shell
$ picocom
$ minicom
$ cutecom
```

* 串口插拔默认无通知，可以增加一些个udev通知来实现

```
ACTION=="add", KERNEL=="ttyUSB*[0-9]", SUBSYSTEM=="tty", RUN+="/sbin/runuser -l $user -c 'notify-send %k 串口已插入'"
ACTION=="remove", KERNEL=="ttyUSB*[0-9]", SUBSYSTEM=="tty", RUN+="/sbin/runuser -l $user -c 'notify-send %k 串口已拔出'"
```

* 文本转数组，命令行一键生成头文件

```shell
$ xxd -i file
```

* kde网络设置中导入openvpn配置

系统设置>连接>添加连接>导入vpn连接

如果.ovpn配置文件中包含tls-crypt字段，还需要在高级设置中对TLS进行设置

* nfs服务配置文件

```shell
$ /etc/exports
```

* mount绑定目录

```shell
$ sudo mount --bind src dest
```
