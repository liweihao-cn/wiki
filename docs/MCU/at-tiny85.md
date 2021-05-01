# AT tiny85

手上有两个叫LipyPad的小电路板，查询发现是个atmel的avr单片机，并且可以使用digispark提供的库在Arduino IDE中进行编程和烧录，似乎挺好玩的样子，大概了解了一下，随后配置一下开发环境，记录一下。

> 由于国内网络环境的原因，Arduino IDE开发板管理中下载索引文件总是失败，所以需要添加代理，在Arduino IDE的网络设置中手动添加一个http的代理，如果你的网络没有问题，那么就不需要这一步。

1. 添加开发板，添加开发板链接**http://digistump.com/package_digistump_index.json**并进行安装

2. 我自己测试的时候发现烧录依赖libusb-0.1的库，但系统默认安装的是1.0的，所以需要使用apt search搜索安装一下libusb-0.1

3. attiny85烧录的固件来自这里[micronucleus](https://github.com/micronucleus/micronucleus.git)，这个固件使用了一个叫[v-usb](https://github.com/obdev/v-usb.git)的库,需要将micronucleus仓库中commandline目录下的49-micronucleus.rules添加到/etc/udev/rules.d中并重启udev服务

以上，没什么意外的话到这里就可以打开一个blink实例进行测试了，在提示连接开发板后插入USB即可下载成功了。

可以参考[digispark wiki](http://digistump.com/wiki/)中的安装及使用帮助文档。