---
description: 定时器的概念及用法介绍。
---

# 定时器

LCUI 的定时器都是在主线程中处理的，这意味着定时器的时间粒度受到帧率的限制，不能小于每帧的停留时间。举个例子：当前帧率为 120 帧每秒，那么时间粒度就是 8.33 毫秒，如果你设置定时器的等待时间是 20 毫秒，那么实际的等待时间会大于等于 25 毫秒，也就在设置定时器后的第三帧时处理这个定时器。这种精确度的定时器能够应付大多数场景，如果你需要更加精确的定时器，我们建议你选择其它定时器库，或自行编码实现。

### 简单的例子

这个例子展示了如何设置和释放定时器：

```c
#include <stdio.h>
#include <LCUI.h>

void OnTimeout(void *arg)
{
    int *timer_id = arg;

    LCUITimer_Free(*timer_id);
    LCUI_Quit();
    printf("timeout\n");
}

void OnInterval(void *arg)
{
    printf("interval\n");
}

int main(void)
{
    int timer_id;

    LCUI_Init();
    timer_id = LCUI_SetInterval(1000, OnInterval, 0);
    LCUI_SetTimeout(5000, OnTimeout, &timer_id);
    return LCUI_Main();
}
```

首先调用 `LCUI_SetInterval()` 设置定时器每隔 1 秒调用一次 `OnInterval()` 函数，并将它返回的定时器标识号赋值给 `timer_id` ，然后调用 `LCUI_SetTimeout()` 设置一个定时器在 5 秒后调用 `OnTimeout()` 函数，并将 `timer_id` 的引用作为参数传给 `OnTimeout()` 函数。

### 待办事项

**重新设计定时器的接口**

改用面向对象模式代替单例模式，示例：

```c
// 当前的接口设计
LCUI_InitTimer();
LCUI_FreeTimer();
LCUITimer_Set();
LCUITimer_Free();
LCUI_ProcessTimers();

// 新的接口设计
TimerList_New();
TimerList_Free();
TimerList_Add();
TimerList_Remove();
TimerList_Process();
```

在新的设计中，不再是全局共用同一个定时器列表，允许开发者基于定时器接口实现一个定时器线程，以此摆脱 LCUI 主循环对定时器精度的影响。
