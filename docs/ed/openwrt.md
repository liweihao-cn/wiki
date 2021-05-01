# OpenWrt

## 1. 克隆

OpenWrt git仓库地址`git://git.openwrt.org/openwrt/openwrt.git`

> 可以使用镜像站点加快克隆，例如`https://gitee.com/harvey520`

## 2. 配置及编译

### 2.1 快速构建

```shell
# 更新软件包，国内网络环境下载可能会比较慢，可以使用镜像站点下载，如下
# 修改 openwrt 源码目录的 feeds.conf.default 文件中的镜像源
# 将 https://git.openwrt.org/feed/packages.git 改为 https://gitee.com/harvey520/packages.git
# 将 https://git.openwrt.org/project/luci.git 改为 https://gitee.com/harvey520/luci.git
# 将 https://git.openwrt.org/feed/routing.git 改为 https://gitee.com/harvey520/routing.git
# 将 https://git.openwrt.org/feed/telephony.git 改为 https://gitee.com/harvey520/telephony.git
./scripts/feeds update -a
./scripts/feeds install -a

# This opens the OpenWrt configuration menu for setting target and options.
# 配置目标板卡
make menuconfig
```

### 2.2 其他选项

```shell
# 打开kernel配置界面
make kernel_menuconfig
```

## 3. 定制

### 3.1 添加自定义文件

> You can include custom files in your image by placing them in <buildroot>/files, e.g. if you want to have my_config included in your image in the directory /etc/config/ ⇒ <buildroot>/files/etc/config/my_config. If the files directory doesn't exist on your buildsystem, then create it.

### 3.2 修改dts

设备树路径是`target/linux/xxx/dts`，例如rt5350f的设备树是`target/linux/ramips/dts/rt5350.dtsi`及相关板卡文件。

### 3.3 内核源码

内核源码路径应该是`build_dir/target-xxx/linux-xxx/linux-x.x.xx`，配置文件的路径是`target/linux/target-name/xxx/config`，例如rt5350f的配置文件是`target/linux/ramips/rt305x/config-4.14`。

### 3.4 udhcpc

**/usr/share/udhcpc/default.script**
**/etc/udhcpc.user**

```shell
root@OpenWrt:/tmp/run# udhcpc --help
BusyBox v1.31.1 () multi-call binary.

Usage: udhcpc [-fbqRB] [-t N] [-T SEC] [-A SEC/-n]
        [-i IFACE] [-s PROG] [-p PIDFILE]
        [-oC] [-r IP] [-V VENDOR] [-F NAME] [-x OPT:VAL]... [-O OPT]...

        -i IFACE        Interface to use (default eth0)
        -s PROG         Run PROG at DHCP events (default /usr/share/udhcpc/default.script)
        -p FILE         Create pidfile
        -B              Request broadcast replies
        -t N            Send up to N discover packets (default 3)
        -T SEC          Pause between packets (default 3)
        -A SEC          Wait if lease is not obtained (default 20)
        -b              Background if lease is not obtained
        -n              Exit if lease is not obtained
        -q              Exit after obtaining lease
        -R              Release IP on exit
        -f              Run in foreground
        -S              Log to syslog too
        -r IP           Request this IP address
        -o              Don't request any options (unless -O is given)
        -O OPT          Request option OPT from server (cumulative)
        -x OPT:VAL      Include option OPT in sent packets (cumulative)
                        Examples of string, numeric, and hex byte opts:
                        -x hostname:bbox - option 12
                        -x lease:3600 - option 51 (lease time)
                        -x 0x3d:0100BEEFC0FFEE - option 61 (client id)
                        -x 14:'"dumpfile"' - option 14 (shell-quoted)
        -F NAME         Ask server to update DNS mapping for NAME
        -V VENDOR       Vendor identifier (default 'udhcp VERSION')
        -C              Don't send MAC as client identifier
Signals:
        USR1    Renew lease
        USR2    Release lease
```
