# FreeRTOS 任务管理

## 1. 动态创建任务

使用 `xTaskCreate` 函数动态创建任务，系统会自动从堆（Heap）中分配任务所需的栈（Stack）和任务控制块（TCB）。

### 函数原型

```c
BaseType_t xTaskCreate( TaskFunction_t pxTaskCode,           // 任务函数
                        const char * const pcName,           // 任务名（字符串）
                        const configSTACK_DEPTH_TYPE usStackDepth, // 栈大小（单位是字，非字节）
                        void * const pvParameters,           // 任务参数
                        UBaseType_t uxPriority,              // 任务优先级
                        TaskHandle_t * const pxCreatedTask );// 任务句柄
```

### 参数详解

*   **pxTaskCode**: 指向任务函数的指针。任务函数通常是一个死循环，不能返回。
*   **pcName**: 任务的描述性名称，主要用于调试，最大长度由 `configMAX_TASK_NAME_LEN` 定义。
*   **usStackDepth**: 任务栈的大小。**注意：单位是 StackType_t（通常是 4 字节），而不是字节。** 例如传入 100，在 32 位系统上分配的是 400 字节。
*   **pvParameters**: 传递给任务函数的参数指针，如果没有参数可传 `NULL`。
*   **uxPriority**: 任务的优先级。数值越大，优先级越高（0 是最低优先级）。
*   **pxCreatedTask**: 用于传出任务句柄。如果不需要句柄，可传 `NULL`。

### 任务句柄 (TaskHandle_t) 的作用

任务句柄是任务的唯一标识，类似于文件句柄或进程 ID。

*   **标识任务**：区分不同的任务实例。
*   **管理任务**：通过句柄调用 API 操作任务：
    *   **删除任务**：`vTaskDelete(pxCreatedTask)`
    *   **挂起任务**：`vTaskSuspend(pxCreatedTask)`
    *   **恢复任务**：`vTaskResume(pxCreatedTask)`
    *   **修改优先级**：`vTaskPrioritySet(pxCreatedTask, newPriority)`
    *   **查询状态**：`eTaskGetState(pxCreatedTask)`
*   **通信机制**：用于向特定任务发送通知（Task Notifications）。

---

## 2. 静态创建任务

使用 `xTaskCreateStatic` 函数静态创建任务，任务的栈和 TCB 内存由用户在编译时或运行时手动分配（通常定义全局数组），不占用动态堆内存。

### 函数原型

```c
TaskHandle_t xTaskCreateStatic( TaskFunction_t pxTaskCode,           // 任务函数
                                const char * const pcName,           // 任务名
                                const uint32_t ulStackDepth,         // 栈大小
                                void * const pvParameters,           // 任务参数
                                UBaseType_t uxPriority,              // 任务优先级
                                StackType_t * const puxStackBuffer,  // 栈缓冲区（用户分配）
                                StaticTask_t * const pxTaskBuffer ); // TCB缓冲区（用户分配）
```

### 注意事项
*   需要手动定义 `StackType_t` 类型的数组作为栈空间。
*   需要手动定义 `StaticTask_t` 类型的变量作为 TCB 空间。
*   相比动态创建，静态创建更安全（无内存碎片风险），但使用相对繁琐。
*   *注：静态任务创建在某些配置下可能存在特定限制或 Bug，需查阅具体版本文档。*

---

## 3. 任务删除

使用 `vTaskDelete` 函数将任务从内核管理列表中移除。

### 函数原型

```c
void vTaskDelete( TaskHandle_t xTaskToDelete );
```

*   **xTaskToDelete**: 要删除的任务句柄。如果传入 `NULL`，则删除调用该函数的任务（即删除自己）。

### 内存回收机制（重要）

1.  **动态创建的任务 (`xTaskCreate`)**:
    *   任务被删除后，其占用的栈和 TCB 空间**不会立即释放**。
    *   这些内存由 **空闲任务 (Idle Task)** 负责回收。
    *   **风险提示**：如果高优先级任务频繁创建和删除任务，导致空闲任务一直得不到执行（Starvation），内存将无法回收，最终导致**内存耗尽**。确保空闲任务有运行机会。

2.  **静态创建的任务 (`xTaskCreateStatic`)**:
    *   系统**不会**自动释放内存，因为内存是用户定义的变量。用户需要自己管理这些变量的生命周期。

---

## 4. 任务调度与优先级

FreeRTOS 是一个抢占式（Preemptive）的实时操作系统。

*   **抢占式调度**：
    *   高优先级任务一旦就绪，会立即抢占低优先级任务的 CPU 使用权。
    *   除非高优先级任务进入阻塞态（Blocked）或挂起态（Suspended），否则低优先级任务无法运行。
*   **时间片轮转**：
    *   当多个任务处于**相同的高优先级**时，它们会轮流执行（时间片轮转）。
*   **注意事项**：
    *   如果有一个高优先级任务通过死循环一直执行（且不调用阻塞延时函数如 `vTaskDelay`），低优先级任务将永远得不到执行机会（被“饿死”）。
