# FreeRTOS的目录结构(基于STM32F1)
## 文件树
```
FreeRTOS/
│
├── Demo/                    # 示例工程
│   ├── Common/
│   └── CORTEX_STM32F103_Keil/
│
├── License/                 # 许可证
│   └── license.txt
│
├── Source/                  # 内核源码
│   ├── include/             # 头文件
│   ├── portable/             # 端口文件
│   ├── croutine.c            # 协程
│   ├── event_groups.c         # 事件组
│   ├── list.c                 # 链表 
│   ├── queue.c                 # 队列
│   ├── stream_buffer.c         # 流缓冲区
│   ├── tasks.c                 # 任务
│   └── timers.c                 # 定时器
│
├── README.md
├── links_to_doc_pages_for_the_demo_projects.url
└── 目录结构.txt
```

