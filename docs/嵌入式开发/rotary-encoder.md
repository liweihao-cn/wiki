# 旋转编码器

## EC11

Linux内核驱动中旋转编码器的状态检测逻辑

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
