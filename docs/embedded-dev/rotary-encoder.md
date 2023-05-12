# 旋转编码器

## EC11

> 注：以下内容均为分析代码得出，未经实际验证。

Linux内核驱动中旋转编码器的状态检测逻辑如下所示

```
rotary-encoder,encoding = "gray"

BIT:  1  0
PIN:  B  A

CW/clockwise/顺时针

A ````````````````````|__________________|````````````

B ```````````|_________________|``````````````````````


raw(gray)   01       00        10        11

GPIO_ACTIVE_LOW
logic(gray) 10       11        01        00

normal      11       10        01        00
state               armed     (2-1)    report


CCW/counter-clockwise/逆时针

A ```````````|_________________|``````````````````````

B ````````````````````|__________________|````````````


raw(gray)   10        00       01        11

GPIO_ACTIVE_LOW
logic(gray) 01        11       10        00

normal      01        10       11        00
state                armed    (2-3)     report

```

内核驱动中将读取到的旋转编码器编码值由格雷码转换为普通二进制数值，利用该数值实现了状态机并进行了事件上报。

内核中读取编码器数值并进行转换的代码如下

```c
static unsigned int rotary_encoder_get_state(struct rotary_encoder *encoder)
{
        int i;
        unsigned int ret = 0;

        for (i = 0; i < encoder->gpios->ndescs; ++i) {
                int val = gpiod_get_value_cansleep(encoder->gpios->desc[i]);

                /* convert from gray encoding to normal */
                if (encoder->encoding == ROTENC_GRAY && ret & 1)
                        val = !val;

                ret = ret << 1 | val;
        }

        return ret & 3;
}
```

其中，将格雷码转换为普通数值时使用了一个简单的`!`逻辑，其原理是因为旋转编码器数值只有两位，当我们将两位的普通数值与格雷码遍历写出时会得到如下内容：

| normal | gray |
| :----: | :--: |
|00|00|
|01|01|
|10|11|
|11|10|

不难看出，当高位为1时低位的状态正好是相反的，因此当高位为1时把低位取反便可以将编码器数值由格雷码转换为普通数值了。
