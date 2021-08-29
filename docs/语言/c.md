# C/C++

## 1. C/C++的相互调用

### 1.1 C++调用C（使用g++进行编译时候的处理）

当统一使用g++进行编译的时候，对于.c文件所对应的.h文件里面的函数声明，应该使用下面的代码快进行包裹以表明该函数为C语言函数。

```c
#if defined(__cplusplus)
extern "C" {
#endif
/* 函数声明 */
#if defined(__cplusplus)
}
#endif
```

### 1.2 C调用C++

C无法直接调用C++函数，必须添加一层wrapper，在该wrapper中对我们需要的功能进行简单封装，最后再以C的形式进行声明，即可实现C调用C++。

```c
/* apds9960_wrapper.h */
#ifndef APP_APDS9960_WRAPPER_H_
#define APP_APDS9960_WRAPPER_H_

#include <stdio.h>

#if defined(__cplusplus)
extern "C" {
#endif

bool apds9960_init(void);
void apds9960_startGesture(void);
uint8_t apds9960_read(void);

#if defined(__cplusplus)
}
#endif

#endif /* APP_APDS9960_WRAPPER_H_ */
```

```c
/* apds9960_wrapper.cpp */
#include "apds9960_wrapper.h"
#include "Adafruit_APDS9960.h"

Adafruit_APDS9960 apds;

bool apds9960_init(void)
{
    return apds.begin();
}

void apds9960_startGesture(void)
{
    apds.enableProximity(true);
    apds.enableGesture(true);
}

uint8_t apds9960_read(void)
{
    return apds.readGesture();
}
```