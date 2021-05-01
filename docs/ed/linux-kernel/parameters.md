# Parameters

<a id="kernel-parameters"></a>

## 1. earlycon

<a id="earlycon"></a>

earlycon在正式的console设备可用之前输出内核的启动信息，earlycon驱动会负责map参数中指定的寄存器地址，map长度为64，详细的参数可以参考kernel-parameters.txt文档。

## 2. initcall_debug

<a id="initcall_debug"></a>

initcall_debug用来追踪initcall调用的进度，有时候有些异常情况会导致initcall中的某些部分阻塞，这个参数对我们的调试很有帮助。
