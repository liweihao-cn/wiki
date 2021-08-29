# wpa_supplicant

wpa_supplicant是跨平台的无线访问客户端，可用于PC、笔记本以及嵌入式等不同场合，详细文档可查看[man手册](https://linux.die.net/man/8/wpa_supplicant)。

## 连接wifi

在嵌入式系统中可按照如下顺序使用wpa_supplicant工具来连接wifi。

* 使能`wlan0`接口

```shell
ifconfig wlan0 up
```

* 热点扫描（可选）

```shell
iw dev wlan0 scan
```

* 使用`wpa_passphrase`生成配置文件

```shell
wpa_passphrase SSID Passwd > /etc/wpa_supplicant.conf
```

* 应用配置

```shell
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
```

* 获取IP

```shell
udhcpc -i wlan0
```

## 参考链接

[https://whycan.com/t_5292.html](https://whycan.com/t_5292.html)
