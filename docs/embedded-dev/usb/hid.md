# HID报告描述符

## 报告描述符相关教程

[Tutorial about USB HID Report Descriptors](https://eleccelerator.com/tutorial-about-usb-hid-report-descriptors/)

[USB HID报告描述符教程](https://zhuanlan.zhihu.com/p/27568561)

## Linux下查看设备HID

> 依赖`usbutils`和`hidrd`软件包，需要自行下载安装。

###  使用debugfs查看

```shell
$ # 直接导出描述符查看
$ cat /sys/kernel/debug/hid/0003\:04F3\:0103.0001/rdesc
05 01 09 06 a1 01 05 08 19 01 29 03 15 00 25 01 75 01 95 03 91 02 95 05 91 01 05 07 19 e0 29 e7 95 08 81 02 75 08 95 01 81 01 19 00 29 91 26 ff 00 95 06 81 00 c0

  INPUT[INPUT]
    Field(0)
      Application(GenericDesktop.Keyboard)
      Usage(8)
        Keyboard.00e0
        Keyboard.00e1
        Keyboard.00e2
        Keyboard.00e3
        Keyboard.00e4
        Keyboard.00e5
        Keyboard.00e6
        Keyboard.00e7
      Logical Minimum(0)
      Logical Maximum(1)
      Report Size(1)
      Report Count(8)
      Report Offset(0)
      Flags( Variable Absolute )
    Field(1)
      Application(GenericDesktop.Keyboard)

  ...

$ # 使用hidrd-convert工具转换格式
$ cat /sys/kernel/debug/hid/0003\:04F3\:0103.0001/rdesc | head -n1 | xxd -r -p | hidrd-convert -o spec
Usage Page (Desktop),               ; Generic desktop controls (01h)
Usage (Keyboard),                   ; Keyboard (06h, application collection)
Collection (Application),
    Usage Page (LED),               ; LEDs (08h)
    Usage Minimum (01h),
    Usage Maximum (03h),
    Logical Minimum (0),
    Logical Maximum (1),
    Report Size (1),
    Report Count (3),
    Output (Variable),
    Report Count (5),
    Output (Constant),
    Usage Page (Keyboard),          ; Keyboard/keypad (07h)
    Usage Minimum (KB Leftcontrol), ; Keyboard left control (E0h, dynamic value)
    Usage Maximum (KB Right GUI),   ; Keyboard right GUI (E7h, dynamic value)
    Report Count (8),
    Input (Variable),
    Report Size (8),
    Report Count (1),
    Input (Constant),
    Usage Minimum (None),           ; No event (00h, selector)
    Usage Maximum (KB LANG2),       ; Keyboard LANG2 (91h, selector)
    Logical Maximum (255),
    Report Count (6),
    Input,
End Collection
```

### 使用usbutils进行查看

```shell
$ # 导出原始格式
$ usbhid-dump -d 04f3 -i 255
006:002:001:DESCRIPTOR         1668584980.090532
 05 0C 09 01 A1 01 85 01 19 00 2A 3C 02 15 00 26
 3C 02 95 01 75 10 81 00 C0 05 01 09 80 A1 01 85
 02 19 81 29 83 15 00 25 01 75 01 95 03 81 02 95
 05 81 01 C0

006:002:000:DESCRIPTOR         1668584980.094501
 05 01 09 06 A1 01 05 08 19 01 29 03 15 00 25 01
 75 01 95 03 91 02 95 05 91 01 05 07 19 E0 29 E7
 95 08 81 02 75 08 95 01 81 01 19 00 29 91 26 FF
 00 95 06 81 00 C0

$ # 使用hidrd-convert工具转换格式
$ usbhid-dump -d 04f3 -i 255 | grep -v : | xxd -r -p | hidrd-convert -o spec
Usage Page (Consumer),              ; Consumer (0Ch)
Usage (Consumer Control),           ; Consumer control (01h, application collection)
Collection (Application),
    Report ID (1),
    Usage Minimum (00h),
    Usage Maximum (AC Format),      ; AC format (023Ch, selector)
    Logical Minimum (0),
    Logical Maximum (572),
    Report Count (1),
    Report Size (16),
    Input,
End Collection,
Usage Page (Desktop),               ; Generic desktop controls (01h)
Usage (Sys Control),                ; System control (80h, application collection)
Collection (Application),
    Report ID (2),
    Usage Minimum (Sys Power Down), ; System power down (81h, one-shot control)
    Usage Maximum (Sys Wake Up),    ; System wake up (83h, one-shot control)
    Logical Minimum (0),
    Logical Maximum (1),
    Report Size (1),
    Report Count (3),
    Input (Variable),
    Report Count (5),
    Input (Constant),
End Collection,
Usage Page (Desktop),               ; Generic desktop controls (01h)
Usage (Keyboard),                   ; Keyboard (06h, application collection)
Collection (Application),
    Usage Page (LED),               ; LEDs (08h)
    Usage Minimum (01h),
    Usage Maximum (03h),
    Logical Minimum (0),
    Logical Maximum (1),
    Report Size (1),
    Report Count (3),
    Output (Variable),
    Report Count (5),
    Output (Constant),
    Usage Page (Keyboard),          ; Keyboard/keypad (07h)
    Usage Minimum (KB Leftcontrol), ; Keyboard left control (E0h, dynamic value)
    Usage Maximum (KB Right GUI),   ; Keyboard right GUI (E7h, dynamic value)
    Report Count (8),
    Input (Variable),
    Report Size (8),
    Report Count (1),
    Input (Constant),
    Usage Minimum (None),           ; No event (00h, selector)
    Usage Maximum (KB LANG2),       ; Keyboard LANG2 (91h, selector)
    Logical Maximum (255),
    Report Count (6),
    Input,
End Collection
```

## Boot Interface Descriptor

USB HID规范中写到 **Set_Protocol Request** 可以在boot protocol和report protocol中进行切换。

一般来说，固件（比如UEFI）中对USB键盘只支持boot protocol，在对USB键盘的初始化流程中会使用 **Set_Protocol Request** 将键盘设置为boot protocol，boot protocol的定义是兼容report protocol的，在HID规范附录B.2中提供了一个描述符可以直接使用，描述符内容如下

```
Usage Page (Generic Desktop),
Usage (Keyboard),
Collection (Application),
  Report Size (1),
  Report Count (8),
  Usage Page (Key Codes),
  Usage Minimum (224),
  Usage Maximum (231),
  Logical Minimum (0),
  Logical Maximum (1),
  Input (Data, Variable, Absolute),
  Report Count (1),
  Report Size (8),
  Input (Constant),
  Report Count (5),
  Report Size (1),
  Usage Page (LEDs),
  Usage Minimum (1),
  Usage Maximum (5),
  Output (Data, Variable, Absolute),
  Report Count (1),
  Report Size (3),
  Output (Constant),
  Report Count (6),
  Report Size (8),
  Logical Minimum (0),
  Logical Maximum(255),
  Usage Page (Key Codes),
  Usage Minimum (0),
  Usage Maximum (255),
  Input (Data, Array),
End Collection
```
