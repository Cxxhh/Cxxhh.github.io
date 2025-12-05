# FreeRTOS 数据类型和编程规范

## 数据类型

### 变量命名前缀

| 变量名前缀 | 含义 |
| :--------- | :--- |
| `c` | `char` |
| `s` | `int16_t`, `short` |
| `l` | `int32_t`, `long` |
| `x` | `BaseType_t`, 其他非标准的类型: 结构体、task handle、queue handle等 |
| `u` | `unsigned` |
| `p` | 指针 (pointer) |
| `uc` | `uint8_t`, `unsigned char` |
| `pc` | `char指针` (char pointer) |

### 函数命名前缀

| 函数名前缀 | 含义 |
| :--------- | :--- |
| `prv` | `private`私有的,带`static`的变量 `prvSetupHardware`|
| `v` | `void`  `vTaskStartScheduler`|
| `x` | `BaseType_t`  同变量命名前缀|

## 宏命名规范

### 宏命名前缀

宏的名字是大小写敏感的，可以添加小写的前缀。前缀是用来表示：宏在哪个文件中定义。

| 宏的前缀 | 示例 | 含义: 在哪个文件里定义 |
| :------- | :--- | :--------------------- |
| `port` | 比如`portMAX_DELAY` | `portable.h`或`portmacro.h` |
| `task` | 比如`taskENTER_CRITICAL()` | `task.h` |
| `pd` | 比如`pdTRUE` | `projdefs.h` |
| `config` | 比如`configUSE_PREEMPTION` | `FreeRTOSConfig.h` |
| `err` | 比如`errQUEUE_FULL` | `projdefs.h` |

### 通用的宏定义

通用的宏定义如下:

| 宏 | 值 |
| :-- | :- |
| `pdTRUE` | `1` |
| `pdFALSE` | `0` |
| `pdPASS` | `1` |
| `pdFAIL` | `0` |
