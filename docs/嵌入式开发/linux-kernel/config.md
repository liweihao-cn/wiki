# 配置

<a id="config"></a>

## 1. 保存配置

<a id="savedefconfig"></a>

添加一个新的板级配置时，可以选用已有的比较相近的其他板卡的配置，通过menuconfig进行配置，修改代码，然后通过`make savedefconfig`来生成配置文件，然后直接将defconfig复制保存即可作为我们新添加的板卡的配置文件。

## 2. tags

<a id="tags"></a>

使用vim查看源码时ctags的跳转功能还是非常实用的，内核提供了一个选项来生成只包含指定架构的ctags索引文件

```shell
make ARCH=arm tags
```
