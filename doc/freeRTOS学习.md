### 
函数原型：
```

```
返回值：
```
成功：
失败：
```
功能：
前置条件：

# freeRTOS API引用 API Reference
## 1.创建任务 Task Creation
### xTaskCreate
函数原型：
```
 BaseType_t xTaskCreate( TaskFunction_t pvTaskCode,  // 任务入口函数指针
                         const char * const pcName,  // 任务名
                         const configSTACK_DEPTH_TYPE uxStackDepth, // 需要分配的任务栈word数
                         void *pvParameters, // 任务入口函数指针入参
                         UBaseType_t uxPriority, // 指定任务优先级
                         TaskHandle_t *pxCreatedTask // 任务句柄
                       );

```
返回值：
```
成功：返回pdPASS
失败：返回errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY
```
功能：创建一项新任务 并将其添加到准备运行的任务列表中。系统自动从freeRTOS堆上分配任务栈
前置条件：`configSUPPORT_DYNAMIC_ALLOCATION`设置为1，或处于未定义状态（默认为 1）
```
内存布局示例：
+-------------------+ 高地址
|  任务栈N (PSP)    | <- 任务N的栈
+-------------------+
|   ...             |
+-------------------+
|  任务栈1 (PSP)    | <- 任务1的栈
+-------------------+
|  Idle任务栈(PSP)  | <- 空闲任务栈
+-------------------+
|  Timer任务栈(PSP) | <- 定时器服务任务栈
+-------------------+
|  ISR栈 (MSP)      | <- 中断/异常专用栈
+-------------------+
|  FreeRTOS堆       | <- FreeRTOS动态内存（pvPortMalloc）
+-------------------+
|    .data (RW)     |
+-------------------+
|     .bss (ZI)     |
+-------------------+
|    .text (RO)     | <- 包含FreeRTOS内核代码
+-------------------+ 低地址
```
### xTaskCreateStatic
函数原型：
```
 TaskHandle_t xTaskCreateStatic( TaskFunction_t pxTaskCode,  // 任务入口函数指针
                                 const char * const pcName,  // 任务名
                                 const uint32_t ulStackDepth, // puxStackBuffer的数组长度
                                 void * const pvParameters, // 任务入口函数指针入参
                                 UBaseType_t uxPriority, // 指定任务优先级
                                 StackType_t * const puxStackBuffer, // 用作任务栈
                                 StaticTask_t * const pxTaskBuffer // 保存任务的TCB
                                );

```
返回值：
```
成功：创建任务，返回句柄
失败：不创建任务，返回NULL
```
功能：创建一项新任务 并将其添加到准备运行的任务列表中。任务栈空间由开发者提供，好处是可以在编译时静态分配
前置条件：`configSUPPORT_STATIC_ALLOCATION`必须设置为 1

### vTaskDelete
函数原型：
```
void vTaskDelete( TaskHandle_t xTask ); // 要删除的任务的句柄，如果传NULL,则删除调用任务
```
返回值：无
功能：从RTOS内核管理中移除任务。要删除的任务将从所有就绪、 阻塞、挂起和事件列表中移除。空闲任务负责释放由RTOS内核分配给已删除任务的内存。因此，如果应用程序调用了`vTaskDelete()`，请务必确保空闲任务获得足够的微控制器处理时间
前置条件：`INCLUDE_vTaskDelete`必须定义为 1

## 2.任务控制 Task Control
### vTaskDelay
函数原型：
```
void vTaskDelay( const TickType_t xTicksToDelay ); // 要阻塞的tick数
```
返回值：无
功能：按给定的tick数延迟任务，任务保持阻塞的实际时间取决于tick频率。实际执行周期等于任务执行时间+延时时间，因此周期不固定，因为任务执行时间可能有变化。
前置条件：`INCLUDE_vTaskDelay`必须定义为1，才可使用此函数

### vTaskDelayUntil
函数原型：
```
void vTaskDelayUntil( TickType_t *pxPreviousWakeTime, // 指向一个变量的指针，该变量用于保存任务最后一次解除阻塞的时间。该变量在首次使用前，必须用当前时间初始化。在这之后，该变量会在vTaskDelayUntil()内自动更新
                      const TickType_t xTimeIncrement ); // 周期时间段
```
返回值：无
功能：将任务延迟到指定时间。它以上次被唤醒的时间为基准，确保任务在固定时间点被唤醒，可以自动补偿任务执行时间，维持固定周期。
前置条件：`INCLUDE_vTaskDelayUntil`必须定义为1，才可使用此函数

### xTaskDelayUntil
函数原型：
```
BaseType_t xTaskDelayUntil( TickType_t *pxPreviousWakeTime, // 指向一个变量的指针，该变量用于保存任务最后一次解除阻塞的时间。该变量在首次使用前，必须用当前时间初始化。在这之后，该变量会在vTaskDelayUntil()内自动更新
                            const TickType_t xTimeIncrement ); // 周期时间段
```
返回值：
可用于检查任务是否实际延时，如果下一个预计唤醒时间已经过去，则任务将不会延时
```
成功：返回pdTRUE
失败：返回pdFALSE
```
功能：将任务延迟到指定时间
前置条件：`INCLUDE_xTaskDelayUntil`必须定义为 1 ，此函数才可用

### uxTaskPriorityGet
函数原型：
```
UBaseType_t uxTaskPriorityGet( const TaskHandle_t xTask ); // 任务句柄
```
返回值：任务的优先级
功能：获取任意任务的优先级，传入NULL会获取调用任务的优先级
前置条件：`INCLUDE_uxTaskPriorityGet`必须定义为 1，才可使用此函数

### vTaskPrioritySet
函数原型：
```
void vTaskPrioritySet( TaskHandle_t xTask,  // 任务句柄，为NULL则设置调用任务的优先级
                       UBaseType_t uxNewPriority ); // 优先级
```
返回值：无
功能：设置任何任务的优先级，如果正在设置的任务的优先级高于当前执行任务的优先级，则函数返回之前将发生上下文切换
前置条件：`INCLUDE_vTaskPrioritySet`必须定义为1，才可使用此函数

### vTaskSuspend
函数原型：
```
void vTaskSuspend( TaskHandle_t xTaskToSuspend ); // 任务句柄，为NULL将导致调用任务被挂起
```
返回值：无
功能：挂起任意任务，无论任务优先级如何，任务被挂起后将永远无法获取任何微控制器处理时间。对该函数的调用次数不会累积，如同一个任务，多次调用`vTaskSuspend`，也只需要调用一次`vTaskResume`即可恢复
前置条件：`INCLUDE_vTaskSuspend`必须定义为1，才可使用此函数

### vTaskResume
函数原型：
```
void vTaskResume( TaskHandle_t xTaskToResume ); // 任务句柄
```
返回值：无
功能：恢复已挂起的任务
前置条件：`INCLUDE_vTaskSuspend`必须定义为1，才可使用此函数

### xTaskResumeFromISR
函数原型：
```
BaseType_t xTaskResumeFromISR( TaskHandle_t xTaskToResume ); // 要恢复的任务句柄
```
返回值：
```
如果恢复任务导致上下文切换，则返回pdTRUE
否则返回pdFALSE。ISR使用此信息来确定ISR之后是否需要上下文切换
```
功能：恢复可以从ISR内调用的挂起的任务
前置条件：`INCLUDE_vTaskSuspend`和`INCLUDE_xTaskResumeFromISR`必须定义为1，才可使用此函数

### xTaskAbortDelay
函数原型：
```
BaseType_t xTaskAbortDelay( TaskHandle_t xTask ); // 将被强制退出阻塞状态的任务的句柄
```
返回值：如果xTask引用的任务不在“阻塞”状态，则返回pdFAIL。否则返回pdPASS。
功能：强制任务离开阻塞状态，并进入“准备就绪”状态，即使任务在阻塞状态下等待的事件没有发生，并且任何指定的超时没有过期
前置条件：`INCLUDE_xTaskAbortDelay`必须定义为1，此函数才可用

### uxTaskPriorityGetFromISR
函数原型：
```
UBaseType_t uxTaskPriorityGetFromISR( const TaskHandle_t xTask ); // 待查询的任务句柄，传递NULL会导致返回调用任务的优先级
```
返回值：xTask的优先级
功能：获取任何任务的优先级。在中断服务程序 (ISR) 中使用此函数是安全的
前置条件：`INCLUDE_uxTaskPriorityGet`必须定义为1，才可使用此函数

### uxTaskBasePriorityGet
函数原型：
```
UBaseType_t uxTaskBasePriorityGet( const TaskHandle_t xTask ); // 待查询任务的句柄。传递NULL会返回调用任务的基础优先级
```
返回值：xTask的基础优先级
功能：获取任意任务的基础优先级。任务的基础优先级是任务当前优先级被继承后将返回的优先级，旨在避免在获取互斥锁时出现无限制的优先级反转。
前置条件：`INCLUDE_uxTaskPriorityGet`和`configUSE_MUTEXES`必须定义为1，才可使用此函数

### uxTaskBasePriorityGetFromISR
函数原型：
```
UBaseType_t uxTaskBasePriorityGetFromISR( const TaskHandle_t xTask ); // 待查询任务的句柄。传递NULL会返回调用任务的基础优先级
```
返回值：xTask的基础优先级
功能：获取任意任务的基础优先级，此函数可以安全地在中断服务程序 (ISR) 中使用
前置条件：`INCLUDE_uxTaskPriorityGet`和`configUSE_MUTEXES`必须定义为1，才可使用此函数

## 3.任务实用程序 Task Utilities
### uxTaskGetSystemState
函数原型：
```
UBaseType_t uxTaskGetSystemState(
                       TaskStatus_t * const pxTaskStatusArray, // TaskStatus_t数组
                       const UBaseType_t uxArraySize, // 数组长度，必须大于等于RTOS控制的任务数量
                       unsigned long * const pulTotalRunTime ); // 目标启动以来的总运行时间，可传入NULL
```
返回值：`TaskStatus_t`数组填充的数量，如果数组长度`uxArraySize`给的不满足条件，则返回0
功能：为系统中的每个任务填充`TaskStatus_t`结构体。使用该函数会导致调度器长时间处于挂起状态，因此该函数仅用于调试
前置条件：`configUSE_TRACE_FACILITY`必须定义为 1，才可使用此函数

### vTaskGetInfo
函数原型：
```
void vTaskGetInfo( TaskHandle_t xTask, // 正在查询的任务的句柄。将 xTask 设置为 NULL 将返回调用任务的信息
                   TaskStatus_t *pxTaskStatus, // 出参
                   BaseType_t xGetFreeStackSpace, // 置pdTRUE表示跳过TaskStatus_t中的高水位线检查
                   eTaskState eState ); // 置eValid表示跳过获取TaskStatus_t中的任务状态信息
```
返回值：无
功能：为单个任务填充`TaskStatus_t`结构体。使用该函数会导致调度器长时间处于挂起状态，因此该函数仅用于调试
前置条件：`configUSE_TRACE_FACILITY`必须定义为1

### xTaskGetApplicationTaskTag / xTaskGetApplicationTaskTagFromISR
函数原型：
```
TaskHookFunction_t xTaskGetApplicationTaskTag( TaskHandle_t xTask ); // 正在查询的任务的句柄。任务可以使用 NULL作为参数值来查询自己的标签值
TaskHookFunction_t xTaskGetApplicationTaskTagFromISR( TaskHandle_t xTask );
```
返回值：正在查询的任务的“标签”值
功能：返回与任务关联的“标签”值。标签值的含义和用途由应用程序编写者定义。RTOS内核本身通常不会访问标签值。
前置条件：`configUSE_APPLICATION_TASK_TAG`必须定义为1，这些函数才可用

### vTaskSetApplicationTaskTag
函数原型：
```
void vTaskSetApplicationTaskTag( TaskHandle_t xTask, // 正在向其分配标签值的任务的句柄
                                 TaskHookFunction_t pxTagValue ); // 正在分配给任务标签的值
```
返回值：无
功能：可为每个任务分配“标签”值。该值仅供应用程序使用，RTOS内核本身不以任何方式使用它
前置条件：`configUSE_APPLICATION_TASK_TAG`必须定义为1，才可使用此函数

### uxTaskGetStackHighWaterMark / uxTaskGetStackHighWaterMark2
函数原型：
```
UBaseType_t uxTaskGetStackHighWaterMark
 ( TaskHandle_t xTask ); // 正在查询的任务的句柄

configSTACK_DEPTH_TYPE uxTaskGetStackHighWaterMark2
 ( TaskHandle_t xTask ); // 正在查询的任务的句柄
```
返回值：以word为单位的高水位标记。如果返回0，则任务可能已经堆栈溢出，接近0则表示即将溢出
功能：返回任务开始执行后任务可用的最小剩余堆栈空间量，即任务堆栈达到最大（最深）值时未使用的堆栈量。这就是所谓的堆栈“高水位线”。随着任务的执行和中断的处理，任务使用的堆栈会增加和缩小。
前置条件：`INCLUDE_uxTaskGetStackHighWaterMark`必须定义为 1，`uxTaskGetStackHighWaterMark`函数才可用。`INCLUDE_uxTaskGetStackHighWaterMark2`必须定义为 1，`uxTaskGetStackHighWaterMark2`函数才可用

### xTaskCallApplicationTaskHook
函数原型：
```
BaseType_t xTaskCallApplicationTaskHook( TaskHandle_t xTask, // 其钩子函数所在任务的句柄
                                         void *pvParameter ); // 要传递给钩子函数的值
```
返回值：
功能：用于调用任务的钩子函数
前置条件：`configUSE_APPLICATION_TASK_TAG` 必须定义为1，此函数才可用

### vTaskSetThreadLocalStoragePointer
函数原型：
```
void vTaskSetThreadLocalStoragePointer( TaskHandle_t xTaskToSet, // 正在写入线程本地数据的任务句柄
                                        BaseType_t xIndex, // 写入数据的线程本地存储数组的索引
                                        void *pvValue ) // 要写入由 xIndex 参数指定的索引的值
```
返回值：无
功能：设置任务的线程本地存储数组中的值。可用数组索引的数量由`configNUM_THREAD_LOCAL_STORAGE_POINTERS`编译时配置常量
前置条件：无

### pvTaskGetThreadLocalStoragePointer
函数原型：
```
void *pvTaskGetThreadLocalStoragePointer(
                                 TaskHandle_t xTaskToQuery, // 正在读取线程本地数据的任务句柄
                                 BaseType_t xIndex ); // 读取数据的线程本地存储数组的索引
```
返回值：存储在任务xTaskToQuery的线程本地存储数组的索引位置xIndex中的值
功能：从任务的线程本地存储数组中检索一个值。可用数组索引的数量由`configNUM_THREAD_LOCAL_STORAGE_POINTERS`编译时配置常量
前置条件：无

### vTaskSetTimeOutState
函数原型：
```
void vTaskSetTimeOutState( TimeOut_t * const pxTimeOut ); // 指向结构体的指针，该结构体将被初始化，用于保存判断超时是否发生所需的信息
```
返回值：无
功能：任务可以进入阻塞状态以等待事件，该函数用于设置初始条件。通常情况下，任务不会无限期地在阻塞状态下等待，而是会指定一个超时时间。如果在任务等待的事件发生之前，超时期限已到，则会解除任务的阻塞状态。
前置条件：无

### xTaskCheckForTimeOut
函数原型：
```
BaseType_t xTaskCheckForTimeOut( TimeOut_t * const pxTimeOut, // 指向结构体的指针，需要在vTaskSetTimeOutState中初始化
                                 TickType_t * const pxTicksToWait ); // 阻塞时间
```
返回值：
```
返回 pdTRUE，表示没有块时间剩余，并且发生超时
返回 pdFALSE，表示保留一些块时间，因此未发生超时
```
功能：检查超时时间
前置条件：无

### eTaskConfirmSleepModeStatus
函数原型：
```
 eSleepModeStatus eTaskConfirmSleepModeStatus( void );
```
返回值：
```
eAbortSleep‌：不能进入睡眠，有任务即将就绪或调度器需立即干预。
‌eStandardSleep‌：可以进入睡眠，但需设置唤醒定时器（因未来有任务会超时唤醒）。
‌eNoTasksWaitingTimeout‌：‌所有任务均处于挂起状态或无限期阻塞状态‌，可‌无限期深度睡眠‌，仅依赖外部中断唤醒
```
功能：用于在进入低功耗（Tickless）模式前确认系统是否可以安全地进入深度睡眠状态，通常在`‌vPortSuppressTicksAndSleep()‌ `函数中被调用，该函数由FreeRTOS内核在满足以下条件时自动触发
前置条件：无

### xTaskGetCurrentTaskHandle
函数原型：
```
TaskHandle_t xTaskGetCurrentTaskHandle( void );
```
返回值：当前正在运行（调用）的任务的句柄
功能：获取当前正在运行的任务的句柄
前置条件：`INCLUDE_xTaskGetCurrentTaskHandle`必须设置为1，才可使用此函数

### xTaskGetIdleTaskHandle
函数原型：
```
TaskHandle_t xTaskGetIdleTaskHandle( void );
```
返回值：与空闲任务关联的任务句柄。RTOS 调度器启动时，自动创建空闲任务
功能：获取空闲任务的句柄
前置条件：`INCLUDE_xTaskGetIdleTaskHandle`必须设置为1，才可使用此函数

### eTaskGetState
函数原型：
```
eTaskState eTaskGetState( TaskHandle_t xTask ); // 查询的任务的句柄
```
返回值：
```
准备就绪	eReady
运行	eRunning（调用任务正在查询自己的优先级）
已阻塞	eBlocked
已挂起	eSuspended
已删除	eDeleted（任务 TCB 正在等待清理）
```
功能：查询任务状态
前置条件：`INCLUDE_eTaskGetState`必须设置为1，才可使用此函数

### pcTaskGetName
函数原型：
```
char * pcTaskGetName( TaskHandle_t xTaskToQuery ); // 所查询任务的句柄，为NULL则查询调用任务的名称
```
返回值：字符串
功能：根据任务的句柄中查找任务的名称
前置条件：无

### xTaskGetHandle
函数原型：
```
TaskHandle_t xTaskGetHandle( const char *pcNameToQuery ); // 名称字符串
```
返回值：找到则返回句柄，否则返回NULL
功能：根据任务的名称查找任务的句柄。此函数需要较长时间才能完成，因此每个任务只能调用一次。获取任务句柄后，需要储存在本地，以供再次使用
前置条件：`INCLUDE_xTaskGetHandle`必须设置为1

### xTaskGetTickCount
函数原型：
```
volatile TickType_t xTaskGetTickCount( void );
```
返回值：自调用`vTaskStartScheduler`以来的tick数
功能：获取自调用`vTaskStartScheduler`以来的tick数，无法从ISR调用此函数
前置条件：无

### xTaskGetTickCountFromISR
函数原型：
```
volatile TickType_t xTaskGetTickCountFromISR( void );
```
返回值：自调用`vTaskStartScheduler`以来的tick数
功能：自调用`vTaskStartScheduler`以来的tick数，可以从ISR调用
前置条件：无

### xTaskGetSchedulerState
函数原型：
```
BaseType_t xTaskGetSchedulerState( void );
```
返回值：
```
taskSCHEDULER_NOT_STARTED  调度器尚未启动。这意味着至少有一个任务已经被创建，但是调度器还没有开始运行
taskSCHEDULER_RUNNING  调度器正在运行。这意味着至少有一个任务正在执行，并且调度器正在按照优先级调度这些任务
taskSCHEDULER_SUSPENDED  调度器已经被挂起。这意味着所有的任务都被暂停了，调度器不再运行。
```
功能：用于获取调度器的状态
前置条件：`INCLUDE_xTaskGetSchedulerState`或`configUSE_TIMERS`必须设置为1，才可使用此函数

### uxTaskGetNumberOfTasks
函数原型：
```
UBaseType_t uxTaskGetNumberOfTasks( void );
```
返回值：RTOS内核当前正在管理的任务数
功能：获取RTOS内核当前正在管理的任务数。这包括所有准备就绪、阻塞和挂起的任务。已删除但尚未被空闲任务释放的任务也将包含在计数中。
前置条件：无

### vTaskList
函数原型：
```
void vTaskList( char *pcWriteBuffer ); // 出参缓冲区
```
返回值：无
功能：调试辅助。`vTaskList()`调用`uxTaskGetSystemState()`，然后将`uxTaskGetSystemState()`生成的原始数据转换为易于阅读的(ASCII) 表格形式，表格中会显示每个任务的状态，其中包括任务的堆栈高水位线
前置条件：`configUSE_TRACE_FACILITY`和`configUSE_STATS_FORMATTING_FUNCTIONS`必须定义为1，才可使用此函数

### vTaskListTasks
函数原型：
```
void vTaskListTasks( char *pcWriteBuffer, size_t uxBufferLength ); // 出参缓冲区，缓冲区长度
```
返回值：无
功能：调试辅助。同上
前置条件：`configUSE_TRACE_FACILITY`和`configUSE_STATS_FORMATTING_FUNCTIONS`必须定义为1，才可使用此函数

### vTaskStartTrace
函数原型：
```
void vTaskStartTrace( char * pcBuffer, unsigned long ulBufferSize ); // 跟踪缓冲区，缓冲区大小
```
返回值：无
功能：启动 RTOS 内核活动跟踪。跟踪记录何时运行任务的标识
前置条件：无

### ulTaskEndTrace
函数原型：
```
unsigned long ulTaskEndTrace( void );
```
返回值：已写入跟踪缓冲区的字节数
功能：停止 RTOS 内核活动跟踪
前置条件：无

### vTaskGetRunTimeStats
函数原型：
```
void vTaskGetRunTimeStats( char *pcWriteBuffer ); // 缓冲区
```
返回值：无
功能：调试辅助。`vTaskGetRunTimeStats()`调用`uxTaskGetSystemState()`，然后将`uxTaskGetSystemState()`生成的原始数据转换为易于阅读的 (ASCII) 表格形式，表格中会显示每个任务在运行状态下所花费的时间（即每个任务消耗的CPU时间量）。数据以绝对值和百分比值的形式提供。绝对值的分辨率取决于应用程序提供的运行时间统计时钟的频率。
前置条件：`configGENERATE_RUN_TIME_STATS`、`configUSE_STATS_FORMATTING_FUNCTIONS`和`configSUPPORT_DYNAMIC_ALLOCATION`必须定义为1，才可使用此函数。此外，应用程序还必须提供`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### vTaskGetRunTimeStatistics
函数原型：
```
void vTaskGetRunTimeStatistics( char *pcWriteBuffer, size_t uxBufferLength ); // 缓冲区和长度
```
返回值：无
功能：调试辅助。`vTaskGetRunTimeStats()`调用`uxTaskGetSystemState()`，然后将`uxTaskGetSystemState()`生成的原始数据转换为易于阅读的 (ASCII) 表格形式，表格中会显示每个任务在运行状态下所花费的时间（即每个任务消耗的CPU时间量）。数据以绝对值和百分比值的形式提供。绝对值的分辨率取决于应用程序提供的运行时间统计时钟的频率。
前置条件：`configGENERATE_RUN_TIME_STATS`、`configUSE_STATS_FORMATTING_FUNCTIONS`和`configSUPPORT_DYNAMIC_ALLOCATION`必须定义为1，才可使用此函数。此外，应用程序还必须提供`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE()`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### vTaskGetIdleRunTimeCounter
函数原型：
```
TickType_t xTaskGetIdleRunTimeCounter( void );
```
返回值：返回空闲任务的运行时间计数器
功能：用于确定空闲任务获得的 CPU 时间
前置条件：`configGENERATE_RUN_TIME_STATS`和`INCLUDE_xTaskGetIdleTaskHandle`必须定义为1，才可使用此函数。此外，应用程序还必须提供`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE()`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### ulTaskGetIdleRunTimeCounter
函数原型：
```
configRUN_TIME_COUNTER_TYPE ulTaskGetIdleRunTimeCounter( void );
```
返回值：返回空闲任务的总运行时间
功能：用于确定空闲任务实际执行的时间
前置条件：`configGENERATE_RUN_TIME_STATS`必须定义为1，才可使用此函数。此外，应用程序还必须提供`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE()`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### ulTaskGetIdleRunTimePercent
函数原型：
```
configRUN_TIME_COUNTER_TYPE ulTaskGetIdleRunTimePercent( void );
```
返回值：返回空闲任务所用 CPU 时间的百分比
功能：用于确定空闲任务所用 CPU 时间的百分比
前置条件：`configGENERATE_RUN_TIME_STATS`必须定义为1，才可使用此函数。此外，应用程序还必须提供`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE()`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### ulTaskGetRunTimeCounter
函数原型：
```
configRUN_TIME_COUNTER_TYPE ulTaskGetRunTimeCounter( const TaskHandle_t xTask ); // 任务句柄
```
返回值：给定任务的总运行时间
功能：获取给定任务的总运行时间
前置条件：`configGENERATE_RUN_TIME_STATS`必须定义为1，才可使用此函数。此外，应用程序还必须提供`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE()`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### ulTaskGetRunTimePercent
函数原型：
```
configRUN_TIME_COUNTER_TYPE ulTaskGetRunTimePercent( const TaskHandle_t xTask ); // 任务句柄
```
返回值：给定任务所用 CPU 时间的百分比
功能：获取给定任务所用 CPU 时间的百分比
前置条件：`configGENERATE_RUN_TIME_STATS`必须定义为1，才可使用此函数。此外，应用程序还必须提供 
`portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()`和`portGET_RUN_TIME_COUNTER_VALUE()`的定义，分别用于配置外设定时器/计数器和返回定时器的当前计数值

### vTaskResetState
函数原型：
```
void vTaskResetState( void );
```
返回值：无
功能：可重置任务模块的内部状态。应用程序必须调用此函数，方可重新启动调度器。
前置条件：无

## 4.RTOS内核控制 Kernel Control
### taskENTER_CRITICAL(), taskEXIT_CRITICAL()
函数原型：
```
void taskENTER_CRITICAL( void );
void taskEXIT_CRITICAL( void );
```
返回值：
功能：通过调用`taskENTER_CRITICAL()`进入临界区，随后 通过调用`taskEXIT_CRITICAL()`退出临界区。其核心作用是‌保护共享资源不被多任务或中断并发访问‌，从而避免数据竞争（Data Race）和程序异常。本质上是通过暂时屏蔽可屏蔽中断，确保临界区内的代码“原子性”执行，不被中断打断。
如果所使用的 FreeRTOS 移植使用了 `configMAX_SYSCALL_INTERRUPT_PRIORITY` 内核配置常量，则调用`taskENTER_CRITICAL()` 会禁用优先级等于或低于 `configMAX_SYSCALL_INTERRUPT_PRIORITY` 设置的优先级的中断，并启用所有高于此优先级的中断，否则全局禁用中断。
临界区必须尽量简短，否则会对中断响应时间产生不利影响。每次调用`taskENTER_CRITICAL()`时，都必须有对应的 `taskEXIT_CRITICAL()`调用。
不得从临界区调用 FreeRTOS API 函数
前置条件：无

### taskENTER_CRITICAL_FROM_ISR(), taskEXIT_CRITICAL_FROM_ISR()
函数原型：
```
UBaseType_t taskENTER_CRITICAL_FROM_ISR( void );
void taskEXIT_CRITICAL_FROM_ISR( UBaseType_t uxSavedInterruptStatus );
```
返回值：
```
taskENTER_CRITICAL_FROM_ISR()返回调用宏之前的中断掩码状态。taskENTER_CRITICAL_FROM_ISR() 返回的值必须作为 uxSavedInterruptStatus 参数用于匹配的 taskEXIT_CRITICAL_FROM_ISR() 调用。
taskEXIT_CRITICAL_FROM_ISR() 不返回任何值。
```
功能：可用于中断服务程序 (ISR) 的`taskENTER_CRITICAL()`and`taskEXIT_CRITICAL()`版本。 每次调用 `taskENTER_CRITICAL_FROM_ISR()`时，都必须有对应的`taskEXIT_CRITICAL_FROM_ISR()`调用。
不得从临界区调用 FreeRTOS API 函数
前置条件：无

### vTaskStartScheduler
函数原型：
```
void vTaskStartScheduler( void );
```
返回值：无
功能：启动 RTOS 调度器。调用后，RTOS 内核可以控制在何时执行哪些任务。空闲任务和定时器守护进程任务（可选）会在 RTOS 调度器启动时自动创建
前置条件：无

### vTaskEndScheduler
函数原型：
```
void vTaskEndScheduler( void );
```
返回值：无
功能：停止 RTOS 内核滴答。所有已创建的任务将自动删除，多任务处理 （无论是抢占式还是协作式）亦将停止。执行将从调用`vTaskStartScheduler()`的位置恢复， 就像`vTaskStartScheduler()`刚刚返回一样。该函数会释放所有由 RTOS 内核分配的资源，但不会释放由应用程序任务分配的资源
前置条件：无

### vTaskSuspendAll
函数原型：
```
void vTaskSuspendAll( void );
```
返回值：无
功能：挂起调度器。挂起调度器会阻止上下文切换， 但会让中断处于启用状态。如果调度器被挂起时，中断请求切换上下文， 那么请求将会被挂起。而且只有在调度器恢复（取消挂起）时才会执行。不得在调度器挂起时调用其他 FreeRTOS API 函数
前置条件：无

### xTaskResumeAll
函数原型：
```
BaseType_t xTaskResumeAll( void );
```
返回值：如果恢复调度器导致了上下文切换，则返回 pdTRUE，否则返回 pdFALSE
功能：恢复通过调用`vTaskSuspendAll()`挂起的调度器。`xTaskResumeAll()`仅恢复调度器，不会恢复之前通过调用 `vTaskSuspend()`而挂起的任务
前置条件：无

### vTaskStepTick
函数原型：
```
 void vTaskStepTick( TickType_t xTicksToJump );
```
返回值：无
功能：？
前置条件：必须将`configUSE_TICKLESS_IDLE`配置常量设置为1

### xTaskCatchUpTicks
函数原型：
```
BaseType_t xTaskCatchUpTicks( TickType_t xTicksToCatchUp ); // 由于中断被禁用而错过的tick中断数。此值不会 自动计算，必须由应用程序编写者自行计算
```
返回值：如果推进tick计数导致任务从阻塞状态中恢复并且发生了上下文切换，则返回`pdTRUE`，否则返回`pdFALSE`。
功能：用于在应用程序代码长时间禁用中断后修正tick计数值
前置条件：无

### taskYIELD
函数原型：无
返回值：无
功能：用于‌主动请求任务切换‌的核心函数。它的主要作用是让当前运行的任务主动放弃 CPU 控制权，促使调度器检查是否有其他更高优先级或同优先级的就绪任务可以运行
前置条件：无

### taskDISABLE_INTERRUPTS()
函数原型：一个用于临时禁用可屏蔽中断的宏
返回值：无
功能：进入“中断禁用状态”，防止中断打断当前任务的执行，从而保证操作的原子性和数据一致性。在使用时，通常会与 `taskENABLE_INTERRUPTS()`配套使用，以恢复中断的使能状态。`taskDISABLE_INTERRUPTS()`仅禁用那些优先级低于或等于 `configMAX_SYSCALL_INTERRUPT_PRIORITY` 的中断。如果中断优先级高于此配置值，则这些中断仍可能触发，此时不应在中断服务程序中调用 FreeRTOS 的 API 函数。此外，在调用`taskDISABLE_INTERRUPTS()`期间，调度器不会被中断触发，当前任务会持续占用 CPU 直到中断被重新启用。因此，在禁用中断期间不能进行任务切换，且应尽量缩短临界区的执行时间以避免影响系统实时性
前置条件：无

### taskENABLE_INTERRUPTS()
函数原型：重新启用被禁用的中断的宏
返回值：无
功能：在调用`taskDISABLE_INTERRUPTS()`禁用中断后，必须使用`taskENABLE_INTERRUPTS()`来重新启用中断，以保证系统的正常运行。
需要注意的是，`taskENABLE_INTERRUPTS()`通常不被直接调用，而是通过`taskENTER_CRITICAL()`和`taskEXIT_CRITICAL()`这两个更高级别的宏来间接使用。这些宏提供了更安全和方便的方式来管理临界区，它们内部会调用 `taskDISABLE_INTERRUPTS()`和`taskENABLE_INTERRUPTS()`来实现中断的禁用和启用
前置条件：无

## 5.RTOS任务通知 Task Notifications
### xTaskNotifyGive 和 xTaskNotifyGiveIndexed
函数原型：
```
 BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify ); // 接收通知的任务句柄

 BaseType_t xTaskNotifyGiveIndexed( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                                    UBaseType_t uxIndexToNotify ); // 通知数组索引
```
返回值：都返回pdPASS
功能：允许任务之间进行简单的信号传递。用于通知任务，通常用于事件通知或信号量释放，而不需要携带任何数据
前置条件：将 configUSE_TASK_NOTIFICATIONS 设置为1 （或保留为未定义状态），才可使用这些宏。常量configTASK_NOTIFICATION_ARRAY_ENTRIES 决定了每项任务的任务通知数组中的索引数

### vTaskNotifyGiveFromISR 和 vTaskNotifyGiveIndexedFromISR
函数原型：
```
void vTaskNotifyGiveFromISR( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                             BaseType_t *pxHigherPriorityTaskWoken ); // 通知是否会触发解阻塞且被解阻塞的任务优先级高于当前任务的标志位，必须初始化为pdFALSE

void vTaskNotifyGiveIndexedFromISR( TaskHandle_t xTaskHandle,  // 接收通知的任务句柄
                                    UBaseType_t uxIndexToNotify,  // 通知数组索引
                                    BaseType_t *pxHigherPriorityTaskWoken ); // 通知是否会触发解阻塞且被解阻塞的任务优先级高于当前任务的标志位，必须初始化为pdFALSE
```
返回值：无
功能：可在中断服务程序 (ISR) 中使用的xTaskNotifyGive 和 xTaskNotifyGiveIndexed版本。如果发送通知导致任务解除阻塞，并且解除阻塞的任务的优先级高于当前正在运行的任务，则*pxHigherPriorityTaskWoken会被设置为pdTRUE。如果vTaskNotifyGiveFromISR将\*pxHigherPriorityTaskWoken设置为true，则应在中断退出前请求上下文切换
前置条件：无

### ulTaskNotifyTake, ulTaskNotifyTakeIndexed
函数原型：
```
uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit, // 退出前是否重置零通知值的标志位
                           TickType_t xTicksToWait ); // 若在该函数调用时未收到通知，在阻塞状态下等待通知的最长时间
   
uint32_t ulTaskNotifyTakeIndexed( UBaseType_t uxIndexToWaitOn,  // 调用任务的通知值数组中的索引
                                  BaseType_t xClearCountOnExit, // 退出前是否重置零通知值的标志位
                                  TickType_t xTicksToWait ); // 若在该函数调用时未收到通知，在阻塞状态下等待通知的最长时间
```
返回值：被递减或重置之前的任务的通知值
功能：用于等待通知。这个函数允许一个任务等待来自另一个任务的通知，从而在需要时切换任务。如果收到 RTOS 任务通知，且xClearCountOnExit设置为pdFALSE，那么 RTOS 任务的通知值将在ulTaskNotifyTake()退出前递减。如果设置为pdTRUE，通知值将在退出前重置为0
前置条件：将 configUSE_TASK_NOTIFICATIONS 设置为 1（或 保留为未定义），这些宏才能可用。常量 configTASK_NOTIFICATION_ARRAY_ENTRIES 设置每个任务的任务通知数组中的索引数

### xTaskNotify 和 xTaskNotifyIndexed
函数原型：
```
 BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                         uint32_t ulValue, // 用于更新目标任务的通知值
                         eNotifyAction eAction ); // 更新目标任务的通知值的方式


 BaseType_t xTaskNotifyIndexed( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                                UBaseType_t uxIndexToNotify, // 目标任务的通知值数组的索引
                                uint32_t ulValue, // 用于更新目标任务的通知值
                                eNotifyAction eAction ); // 更新目标任务的通知值的方式
```
返回值：除了 eAction 设置为 eSetValueWithoutOverwrite 且目标任务的通知值无法更新（因为目标任务已有挂起的通知）时，其他情况下均返回 pdPASS
功能：用于直接向 RTOS 任务发送事件，并且可能解除该任务的阻塞状态， 同时还可以根据eAction的值以以下方式更新接收任务的某个通知值，
```
eNoAction：不对任务的通知值进行任何操作。
eSetBits：将通知值与任务的通知值进行位或（OR）操作。
eIncrement：将任务的通知值增加1。
eSetValueWithOverwrite：直接将通知值设置为新的值，覆盖掉之前的值。
eSetValueWithoutOverwrite：仅当任务的通知值为0时，才设置新的通知值
```
前置条件：将 configUSE_TASK_NOTIFICATIONS 设置为 1 （或保留为未定义状态），才可使用这些函数。常量 configTASK_NOTIFICATION_ARRAY_ENTRIES 决定了每项任务的任务通知数组中的索引数

### xTaskNotifyFromISR 和 xTaskNotifyIndexedFromISR
函数原型：
```
 BaseType_t xTaskNotifyFromISR( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                                uint32_t ulValue, // 用于更新目标任务的通知值
                                eNotifyAction eAction, // 更新目标任务的通知值的方式
                                BaseType_t *pxHigherPriorityTaskWoken ); // 通知是否会触发解阻塞且被解阻塞的任务优先级高于当前任务的标志位，必须初始化为pdFALSE

 BaseType_t xTaskNotifyIndexedFromISR( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                                       UBaseType_t uxIndexToNotify, // 目标任务的通知值数组的索引
                                       uint32_t ulValue, // 用于更新目标任务的通知值
                                       eNotifyAction eAction, // 更新目标任务的通知值的方式
                                       BaseType_t *pxHigherPriorityTaskWoken ); // 通知是否会触发解阻塞且被解阻塞的任务优先级高于当前任务的标志位，必须初始化为pdFALSE
```
返回值：除了eAction设置为eSetValueWithoutOverwrite且目标任务的通知值无法更新（因为目标任务已有挂起的通知）时，其他情况下均返回pdPASS
功能：可在中断服务程序 (ISR) 中使用的 xTaskNotify() 和 xTaskNotifyIndexed() 版本
前置条件：无

### xTaskNotifyAndQuery 和 xTaskNotifyAndQueryIndexed
函数原型：
```
 BaseType_t xTaskNotifyAndQuery( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                                 uint32_t ulValue, // 用于更新目标任务的通知值
                                 eNotifyAction eAction, // 更新目标任务的通知值的方式
                                 uint32_t *pulPreviousNotifyValue ); // 目标任务之前的通知值

 BaseType_t xTaskNotifyAndQueryIndexed( TaskHandle_t xTaskToNotify, // 接收通知的任务句柄
                                        UBaseType_t uxIndexToNotify, // 目标任务的通知值数组的索引
                                        uint32_t ulValue, // 用于更新目标任务的通知值
                                        eNotifyAction eAction, // 更新目标任务的通知值的方式
                                        uint32_t *pulPreviousNotifyValue ); // 目标任务之前的通知值
```
返回值：除了 eAction 设置为 eSetValueWithoutOverwrite 且目标任务的通知值无法更新（因为目标任务已有挂起的通知）时，其他情况下 均返回 pdPASS。
功能：和xTaskNotify相比多了一个出参pulPreviousNotifyValue，用于获取目标任务之前的通知值。不得从中断服务程序 (ISR) 调用此函数
前置条件：无

### xTaskNotifyAndQueryFromISR 和 xTaskNotifyAndQueryIndexedFromISR
函数原型：
```
 BaseType_t xTaskNotifyAndQueryFromISR(
                      TaskHandle_t xTaskToNotify,
                      uint32_t ulValue,
                      eNotifyAction eAction,
                      uint32_t *pulPreviousNotifyValue,
                      BaseType_t *pxHigherPriorityTaskWoken );

 BaseType_t xTaskNotifyAndQueryIndexedFromISR(
                      TaskHandle_t xTaskToNotify,
                      UBaseType_t uxIndexToNotify
                      uint32_t ulValue,
                      eNotifyAction eAction,
                      uint32_t *pulPreviousNotifyValue,
                      BaseType_t *pxHigherPriorityTaskWoken );
```
返回值：除了 eAction 设置为 eSetValueWithoutOverwrite 且目标任务的通知值无法更新（因为目标任务已有挂起的通知）时，其他情况下 均返回 pdPASS。
功能：可在中断服务程序 (ISR) 中使用的xTaskNotifyAndQuery和xTaskNotifyAndQueryIndexed版本
前置条件：无

### xTaskNotifyWait 和 xTaskNotifyWaitIndexed
函数原型：
```
 BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry, // 进入该函数前需要清零的比特域段
                             uint32_t ulBitsToClearOnExit, // 退出该函数前需要清零的比特域段
                             uint32_t *pulNotificationValue, // 用于传出之前的通知值
                             TickType_t xTicksToWait ); // 调用该函数时若没有挂起通知，在阻塞状态下等待接收通知的最长时间

 BaseType_t xTaskNotifyWaitIndexed( UBaseType_t uxIndexToWaitOn, // 调用任务的通知值数组中用于等待接收通知的索引
                                    uint32_t ulBitsToClearOnEntry, // 进入该函数前需要清零的比特域段
                                    uint32_t ulBitsToClearOnExit, // 退出该函数前需要清零的比特域段
                                    uint32_t *pulNotificationValue, // 用于传出之前的通知值
                                    TickType_t xTicksToWait ); // 调用该函数时若没有挂起通知，在阻塞状态下等待接收通知的最长时间
```
返回值：
```
如果收到了通知，或者在调用 xTaskNotifyWait() 时通知已挂起， 则返回 pdTRUE。
如果调用 xTaskNotifyWait() 超时且在超时前没有收到通知， 则返回 pdFALSE
```
功能：允许一个任务等待来自另一个任务或中断服务例程的通知。不能在中断服务程序中调用
前置条件：必须在 FreeRTOSConfig.h 中将 configUSE_TASK_NOTIFICATIONS 设置为 1 （或保留为未定义状态），才可使用这些宏。常量 configTASK_NOTIFICATION_ARRAY_ENTRIES 决定了每项任务的任务通知数组中的索引数

### xTaskNotifyStateClear, xTaskNotifyStateClearIndexed
函数原型：
```
BaseType_t xTaskNotifyStateClear( TaskHandle_t xTask ); // 将清除其通知状态的 RTOS 任务的句柄

BaseType_t xTaskNotifyStateClearIndexed( TaskHandle_t xTask,  // 将清除其通知状态的 RTOS 任务的句柄
                                         UBaseType_t uxIndexToClear ); // 通知值数组索引
```
返回值：
```
如果 xTask 引用的任务有挂起的通知，则通知 已清除，然后返回 pdTRUE。如果 xTask 引用的任务 有待处理的通知，那么返回 pdFALSE。
```
功能：用于清除一个任务的通知状态。当一个任务通过调用 xTaskNotifyGive、ulTaskNotifyTake 或类似的通知函数与另一个任务进行通信时，通知状态可能会被设置或更新，该函数可以清除通知状态。
前置条件：configUSE_TASK_NOTIFICATIONS 必须在 FreeRTOSConfig.h 中设置为 1（或保留为未定义）才能使用这些宏。常量 configTASK_NOTIFICATION_ARRAY_ENTRIES 设置每个任务的任务通知数组中的索引数

### ulTaskNotifyValueClear, ulTaskNotifyValueClearIndexed
函数原型：
```
uint32_t ulTaskNotifyValueClear( TaskHandle_t xTask,  // 将清除其通知值的 RTOS 任务的句柄
                                 uint32_t ulBitsToClear ); // 需要清除的通知值的位掩码
  
uint32_t ulTaskNotifyValueClearIndexed( TaskHandle_t xTask,   // 将清除其通知值的 RTOS 任务的句柄
                                        UBaseType_t uxIndexToClear, // 通知值数组索引
                                        uint32_t ulBitsToClear ); // 需要清除的通知值的位掩码
```
返回值：清除前目标任务的通知值
功能：用于清除一个任务的通知值
前置条件：configUSE_TASK_NOTIFICATIONS 必须在 FreeRTOSConfig.h 中设置为 1（或保留为未定义）才能使用这些宏。 常量 configTASK_NOTIFICATION_ARRAY_ENTRIES 设置 每个任务的任务通知数组中的索引数

## 6.队列 Queues
### xQueueCreate
函数原型：
```
 QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength, // 队列一次可存储的最大项目数
                             UBaseType_t uxItemSize ); // 队列中每个项目所需的大小（以字节为单位）
```
返回值：
```
成功：返回创建队列的句柄
失败：返回NULL
```
功能：创建新队列并返回一个可以引用该队列的句柄。如果使用xQueueCreate()创建队列，则所需的 RAM 会自动从 FreeRTOS 堆中分配。 如果使用 xQueueCreateStatic() 创建队列， 则 RAM 由应用程序编写者提供，这会产生更多的参数，但这样能够在编译时静态分配 RAM
前置条件： configSUPPORT_DYNAMIC_ALLOCATION 必须设置为 1，或保留为未定义状态（默认为 1）， 才可使用此函数

### xQueueCreateStatic
函数原型：
```
 QueueHandle_t xQueueCreateStatic(
                             UBaseType_t uxQueueLength, // 队列一次可存储的最大项目数
                             UBaseType_t uxItemSize, // 队列中每个项目所需的大小（以字节为单位）
                             uint8_t *pucQueueStorageBuffer, // 用户分配的队列空间，大小必须大于等于队列的总字节数
                             StaticQueue_t *pxQueueBuffer ); // 用于保存队列的数据结构体
```
返回值：
```
成功：返回创建队列的句柄
失败：返回NULL
```
功能：创建新队列
前置条件： configSUPPORT_STATIC_ALLOCATION 必须设置为 1，才可使用此 RTOS API 函数

### xQueueSend
函数原型：
```
 BaseType_t xQueueSend(
                        QueueHandle_t xQueue, // 指向队列的句柄
                        const void * pvItemToQueue, // 要发送到队列的数据的指针
                        TickType_t xTicksToWait // 等待队列有空间的时间长度
                      );
```
返回值：
```
pdPASS‌：数据已成功发送到队列。
‌pdFAIL‌：如果调用时没有设置等待时间（xTicksToWait为0），且队列已满，则返回此值。
‌errQUEUE_FULL‌：如果调用了等待时间（xTicksToWait非0），且在等待时间内队列仍然满，则返回此值。
```
功能：用来向队列中发送数据，尾插，队列是同步对象，用于在不同任务之间传递数据
前置条件：

### xQueueSendFromISR
函数原型：
```
 BaseType_t xQueueSendFromISR
           (
               QueueHandle_t xQueue, // 指向队列的句柄
               const void *pvItemToQueue, // 要发送到队列的数据的指针
               BaseType_t *pxHigherPriorityTaskWoken // 是否会解阻塞更高优先级任务
           );
```
返回值：
```
如果数据成功发送至队列，则返回 pdTRUE，
否则返回 errQUEUE_FULL
```
功能：尾插，如果发送到队列会导致任务解除阻塞，并且解除阻塞的任务的优先级高于当前正在运行的任务，则xQueueSendFromISR()会将*pxHigherPriorityTaskWoken设置为pdTRUE。如果xQueueSendFromISR()将此值设置为pdTRUE，则应在中断退出前请求上下文切换
前置条件：无

### xQueueSendToBack
函数原型：
```
 BaseType_t xQueueSendToBack(
        QueueHandle_t xQueue,
        const void * pvItemToQueue,
        TickType_t xTicksToWait
 );
```
返回值：等同于xQueueSend
功能：等同于xQueueSend
前置条件：等同于xQueueSend

### xQueueSendToBackFromISR
函数原型：
```
 BaseType_t xQueueSendToBackFromISR
 (
      QueueHandle_t xQueue,
      const void *pvItemToQueue,
      BaseType_t *pxHigherPriorityTaskWoken
 );

```
返回值：等同于xQueueSendFromISR
功能：等同于xQueueSendFromISR
前置条件：等同于xQueueSendFromISR

### xQueueSendToFront
函数原型：
```
 BaseType_t xQueueSendToFront( QueueHandle_t xQueue, // 指向队列的句柄
 const void * pvItemToQueue, // 要发送到队列的数据的指针
 TickType_t xTicksToWait ); // 等待队列有空间的时间长度
```
返回值：如果成功发布项目，返回 pdTRUE，否则返回 errQUEUE_FULL
功能：头插一个数据到队列，不得从中断服务程序 调用此函数
前置条件：无

### xQueueSendToFrontFromISR
函数原型：
```
 BaseType_t xQueueSendToFrontFromISR
 (
 QueueHandle_t xQueue,
 const void *pvItemToQueue,
 BaseType_t *pxHigherPriorityTaskWoken
 );
```
返回值：如果数据成功发送至队列，则返回 pdPass，否则返回 errQUEUE_FULL
功能：头插
前置条件：无

### xQueueReceive
函数原型：
```
BaseType_t xQueueReceive(
                          QueueHandle_t xQueue, // 指向队列的引用
                          void *pvBuffer, // 用于存储从队列中接收的数据。变量的类型应与队列中存储的数据类型相匹配
                          TickType_t xTicksToWait // 指定任务在返回之前应等待的最长ticks数
);
```
返回值：
```
如果从队列中成功接收项目，则返回 pdTRUE；
否则返回 pdFALSE。
```
功能：用于从队列中接收数据
前置条件：无

### xQueueReceiveFromISR
函数原型：
```
 BaseType_t xQueueReceiveFromISR
           (
               QueueHandle_t xQueue,
               void *pvBuffer,
               BaseType_t *pxHigherPriorityTaskWoken
           );
```
xQueueReceive的ISR版本

### xQueueOverwrite
函数原型：
```
 BaseType_t xQueueOverwrite(
 QueueHandle_t xQueue, // 队列引用
 const void * pvItemToQueue // 指向待入队列的数据
 );
```
返回值：如果成功发布项目，返回 pdTRUE，否则返回 errQUEUE_FULL
功能：即使队列已满，也要覆盖式尾插
前置条件：无

### xQueueOverwriteFromISR
函数原型：
```
BaseType_t xQueueOverwrite
(
    QueueHandle_t xQueue,
    const void * pvItemToQueue
    BaseType_t *pxHigherPriorityTaskWoken
);
```
xQueueOverwrite的ISR版本

### xQueuePeek
函数原型：
```
 BaseType_t xQueuePeek(
 QueueHandle_t xQueue, // 指向队列的引用
 void *pvBuffer, // 用于存储从队列中接收的数据。变量的类型应与队列中存储的数据类型相匹配
 TickType_t xTicksToWait // 指定任务在返回之前应等待的最长ticks数
 );
```
返回值：如果从队列中成功接收，则返回 pdTRUE，否则返回 pdFALSE
功能：从队列中接收项目，但不删除该项目
前置条件：无

### xQueuePeekFromISR
函数原型：
```
BaseType_t xQueuePeekFromISR(
                              QueueHandle_t xQueue,
                              void *pvBuffer,
                             );
```
xQueuePeek的ISR版本

### vQueueAddToRegistry
函数原型：
```
void vQueueAddToRegistry(
                          QueueHandle_t xQueue, // 队列句柄
                          char *pcQueueName, // 为队列指定的名称
                        );
```
返回值：无
功能：调试函数。允许将队列添加到队列注册表中
前置条件：无

### vQueueUnregisterQueue
函数原型：
```
 void vQueueUnregisterQueue( QueueHandle_t xQueue );
```
返回值：无
功能：调试函数。允许将队列从队列注册表中移除
前置条件：无

### pcQueueGetName
函数原型：
```
const char *pcQueueGetName( QueueHandle_t xQueue )
```
返回值：如果 xQueue 引用的队列在队列注册表中，则返回队列的文本名称，否则返回 NULL
功能：根据队列的句柄查找队列名称。队列只有添加到队列注册表时才有名称
前置条件：无

### xQueueGetStaticBuffers
函数原型：
```
 BaseType_t xQueueGetStaticBuffers( QueueHandle_t xQueue, // 要检索其数据结构体缓冲区和存储区缓冲区的队列
                                    uint8_t ** ppucQueueStorage, // 用于返回指向队列存储区缓冲区的指针
                                    StaticQueue_t ** ppxStaticQueue ); // 用于返回指向队列数据结构体缓冲区的指针
```
返回值：
```
如果检索到缓冲区，则返回pdTRUE，否则返回pdFALSE
```
功能：
前置条件：configSUPPORT_STATIC_ALLOCATION必须定义为 1，才可使用此函数

### uxQueueMessagesWaiting
函数原型：
```
UBaseType_t uxQueueMessagesWaiting( QueueHandle_t xQueue ); // 待查询的队列
```
返回值：返回队列中存储的消息数
功能：返回队列中存储的消息数
前置条件：无

### uxQueueMessagesWaitingFromISR
函数原型：
```
UBaseType_t uxQueueMessagesWaiting( QueueHandle_t xQueue ); // 待查询的队列
```
返回值：返回队列中存储的消息数
功能：返回队列中存储的消息数，可以从 ISR 中调用
前置条件：无

### uxQueueSpacesAvailable
函数原型：
```
UBaseType_t uxQueueSpacesAvailable( QueueHandle_t xQueue );
```
返回值：返回队列中的可用空间数
功能：返回队列中的可用空间数
前置条件：无

### vQueueDelete
函数原型：
```
void vQueueDelete( QueueHandle_t xQueue );
```
返回值：删除队列
功能：删除队列，并释放所有内存
前置条件：无

### xQueueReset
函数原型：
```
BaseType_t xQueueReset( QueueHandle_t xQueue );
```
返回值：总是返回 pdPASS
功能：将队列重置为原始的空状态
前置条件：无

### xQueueIsQueueEmptyFromISR
函数原型：
```
BaseType_t xQueueIsQueueEmptyFromISR( const QueueHandle_t pxQueue );
```
返回值：
```
如果队列不为空，则返回 pdFALSE
如果队列为空，则返回 pdTRUE
```
功能：查询队列是否为空，只能用于ISR
前置条件：无

### xQueueIsQueueFullFromISR
函数原型：
```
BaseType_t xQueueIsQueueFullFromISR( const QueueHandle_t pxQueue );
```
返回值：
```
如果队列未满，则返回 pdFALSE；
如果队列已满，则返回 pdTRUE
```
功能：查询队列是否已满
前置条件：无

## 7.队列集 Queue Sets
### xQueueCreateSet
函数原型：
```
QueueSetHandle_t xQueueCreateSet
                 (
                     const UBaseType_t uxEventQueueLength // 队列集可容纳的最大对象数量
                 );
```
返回值：
```
成功返回队列集句柄
失败返回NULL
```
功能：用于创建队列集合。队列集和是一种特殊的内核对象，允许一个任务同时等待多个队列或信号量，当集合中任意一个队列/信号量有数据可用时，等待的任务会被唤醒。这一特性解决了单个任务需要响应多个事件源的场景，避免任务通过轮询多个对象浪费CPU资源的问题。在添加到队列集时，队列和信号量必须为空
前置条件：必须将 configUSE_QUEUE_SETS 设置为 1

### xQueueAddToSet
函数原型：
```
 BaseType_t xQueueAddToSet
                      (
                          QueueSetMemberHandle_t xQueueOrSemaphore, // 队列/信号量句柄
                          QueueSetHandle_t xQueueSet // 队列集句柄
                      );
```
返回值：
```
成功添加到队列集，则返回pdPASS，否则返回pdFAIL
```
功能：将队列/信号量添加至队列集
前置条件：必须将 configUSE_QUEUE_SETS 设置为 1

### xQueueRemoveFromSet
函数原型：
```
BaseType_t xQueueRemoveFromSet
                      (
                          QueueSetMemberHandle_t xQueueOrSemaphore, // 队列/信号量句柄
                          QueueSetHandle_t xQueueSet // 队列集句柄
                      );
```
返回值：
```
成功删除返回pdPASS，不存在或队列不为空返回pdFAIL
```
功能：从队列集中删除 RTOS 队列或信号量。仅当队列或信号量为空时，才能从队列集中删除 RTOS 队列或信号量 
前置条件：必须将 configUSE_QUEUE_SETS 设置为 1

### xQueueSelectFromSet
函数原型：
```
 QueueSetMemberHandle_t xQueueSelectFromSet
                       (
                             QueueSetHandle_t xQueueSet, // 队列集
                             const TickType_t xTicksToWait // 等待时间
                        );
```
返回值：队列集中包含数据的队列的句柄
功能：从队列集的成员中选择包含数据的队列或可用的信号量
前置条件：必须将 configUSE_QUEUE_SETS 设置为 1

### xQueueSelectFromSetFromISR
函数原型：
```
 QueueSetMemberHandle_t xQueueSelectFromSetFromISR
                       (
                             QueueSetHandle_t xQueueSet
                        );
```
返回值：队列集中包含数据的队列的句柄
功能：从队列集的成员中选择包含数据的队列或可用的信号量，可以从中断服务程序 (ISR) 中使用
前置条件：必须将 configUSE_QUEUE_SETS 设置为 1

## 8.流缓冲区 Stream Buffers
将 FreeRTOS/source/stream_buffer.c 源文件包含在构建中即可启用流缓冲区功能
```
xStreamBufferCreate() // 用于‌动态创建流缓冲区（Stream Buffer）‌的函数，适用于任务间或中断与任务间传递‌连续字节流‌的场景

xStreamBufferCreateStatic() // 创建流缓冲区，用户分配空间

xStreamBufferSend() // 将字节发送到流缓冲区

xStreamBufferSendFromISR() // 从 中断服务程序 (ISR) 写入流缓冲区

xStreamBufferReceive() // 从流缓冲区接收字节

xStreamBufferReceiveFromISR() // 从 流缓冲区中接收字节的 API 函数的中断安全版本

vStreamBufferDelete() // 删除之前创建的流缓冲区

xStreamBufferBytesAvailable() // 查询流缓冲区，查看它包含多少数据

xStreamBufferSpacesAvailable() // 查询流缓冲区，查看有多少可用空间

xStreamBufferSetTriggerLevel() // 设置在流缓冲区中被阻塞以等待数据的任务离开阻塞状态之前，流缓冲区中必须有的字节数

xStreamBufferReset() // 将流缓冲区重置为其初始空状态。任何在流缓冲区的数据都将被丢弃。只有当没有任务被阻塞以等待向流缓冲区发送或从流缓冲区接收时， 流缓冲区才能被重置

xStreamBufferResetFromISR() // xStreamBufferReset() 函数的中断安全版本

xStreamBufferIsEmpty() // 查询流缓冲区以查看其是否为空。如果流缓冲区不包含任何数据，则为空

xStreamBufferIsFull() // 查询流缓冲区以查看其是否已满。如果一个流缓冲区没有任何可用空间，则该流缓冲区已满

xStreamBufferGetStaticBuffers() // 检索指向静态创建的流缓冲区数据结构体缓冲区和存储区域缓冲区的指针

uxStreamBufferGetStreamBufferNotificationIndex() // 用于获取与特定StreamBuffer关联的通知索引

vStreamBufferSetStreamBufferNotificationIndex() // 设置所提供的流缓冲区使用的任务通知索引

xStreamBatchingBufferCreate() // 创建新的流批处理缓冲区

xStreamBatchingBufferCreateStatic() // 使用静态分配的内存创建一个新的流批处理缓冲区
```

## 9.消息缓冲区 Message Buffers

```
xMessageBufferCreate() // 用于‌动态创建消息缓冲区，适用于任务间或任务与中断间传递‌可变长度、有边界的离散消息

xMessageBufferCreateStatic() // 使用静态分配的内存创建新的消息缓冲区

xMessageBufferSend() // 将离散消息发送到消息缓冲区。消息可以是适合缓冲区可用空间的任意长度

xMessageBufferSendFromISR() // 中断安全版本的xMessageBufferSend

xMessageBufferReceive() // 从 RTOS 消息缓冲区接收离散消息。消息长度可变，并且从缓冲区中复制出来

xMessageBufferReceiveFromISR() // 中断安全版本的xMessageBufferReceive

vMessageBufferDelete() // 删除之前创建的消息缓冲区

xMessageBufferSpacesAvailable() // 查询一个消息缓冲区还有多少空闲空间

xMessageBufferReset() // 重置消息缓冲区，使其恢复到初始空状态。消息缓冲区中的任何数据都将被丢弃

xMessageBufferResetFromISR() // 中断安全版本的xMessageBufferReset

xMessageBufferIsEmpty() // 查询消息缓冲区是否为空。如果消息缓冲区不包含任何消息，则为空

xMessageBufferIsFull() // 查询消息缓冲区以查看其是否已满。如果消息缓冲区无法再接受任何大小的消息，则消息缓冲区已满，直到通过从消息缓冲区中删除消息来提供空间为止

xMessageBufferGetStaticBuffers() // 检索指向静态创建的消息缓冲区的数据结构体缓冲区和存储区缓冲区的指针
```
## 10.信号量和锁 Semaphore and Mutexes
在许多使用场景中，使用任务通知notify要比使用二进制信号量的速度更快，内存效率更高。
```
xSemaphoreCreateBinary // 创建一个二进制信号量，并返回一个可以引用该信号量的句柄

xSemaphoreCreateBinaryStatic // 创建一个二进制信号量，并返回一个可以引用该信号量的句柄

vSemaphoreCreateBinary // 旧API，使用 xSemaphoreCreateBinary() 函数代替

xSemaphoreCreateCounting // 创建一个计数信号量，并返回一个可以引用该新建信号量的句柄

xSemaphoreCreateCountingStatic // 创建一个计数信号量，并返回一个可以引用该新建信号量的句柄

xSemaphoreCreateMutex // 创建互斥锁，并返回一个该互斥锁可以引用的句柄。中断服务例程中，不能使用互斥锁

xSemaphoreCreateMutexStatic // 创建互斥锁，并返回一个该互斥锁可以引用的句柄。中断服务例程中，不能使用互斥锁

xSemaphoreCreateRecursiveMutex // 创建一个递归互斥锁，并返回一个可以引用该互斥锁的句柄。不能在中断服务程序中使用递归互斥锁

xSemaphoreCreateRecursiveMutexStatic // 创建一个递归互斥锁，并返回一个可以引用该互斥锁的句柄。不能在中断服务程序中使用递归互斥锁

vSemaphoreDelete // 删除信号量，包括互斥锁型信号量和递归信号量

xSemaphoreGetMutexHolder // 获取互斥锁持有者的任务句柄

xSemaphoreTake // 获取信号量，不得从 ISR 调用此宏

xSemaphoreTakeFromISR // 可从 ISR 调用的 xSemaphoreTake() 版本

xSemaphoreTakeRecursive // 递归地获得或“获取”一个互斥锁型信号量

xSemaphoreGive // 释放信号量，不得在 ISR 中使用此宏

xSemaphoreGiveRecursive // 递归地释放或“给出”一个互斥锁型信号量

xSemaphoreGiveFromISR // 释放信号量，可在 ISR 中使用此宏

uxSemaphoreGetCount // 获取信号量计数

```

## 11.软件定时器 Software Timers
软件定时器允许设置一段时间，当设置的时间到达之后就执行指定的功能函数，这个功能函数叫做定时器的回调函数，两次执行回调函数的时间间隔称为定时周期。
这些 API 函数仅在已构建项目中包含'timers.c'源文件，并且在将configUSE_TIMERS设置为1时才可用
```
xTimerCreate // 创建一个新的软件定时器实例，并返回一个可以引用定时器的句柄

xTimerCreateStatic // 创建一个新的软件定时器实例，并返回一个可以引用定时器的句柄

xTimerIsTimerActive // 查询软件定时器是否处于活动或休眠状态

pcTimerGetName // 获取定时器的名称，名称在创建定时器时被分配

vTimerSetReloadMode // 将软件定时器的“模式”更新为自动重新加载定时器或一次性定时器

xTimerStart // 启动定时器，如定时器已启动，则重置定时器

xTimerStartFromISR // 可从中断服务例程调用的 xTimerStart() 的版本

xTimerStop // 停用定时器

xTimerStopFromISR // 可从中断服务例程调用的 xTimerStop() 的版本

xTimerChangePeriod // 改变先前使用 xTimerCreate() 函数创建的定时器的周期，更改休眠定时器的周期也会启动定时器

xTimerChangePeriodFromISR // 可从中断服务例程调用的 xTimerChangePeriod() 的版本

xTimerDelete // 删除之前使用 xTimerCreate() 函数创建的定时器

xTimerReset // 重启之前使用 xTimerCreate() 函数创建的定时器

xTimerResetFromISR // 可从中断服务例程调用的 xTimerReset() 的版本

vTimerResetState // 重置定时器模块的内部状态。在重新启动调度器之前，应用程序必须调用它

pvTimerGetTimerID // 返回分配给软件定时器的 ID

vTimerSetTimerID // 设置定时器ID。创建定时器时会分配一个ID，这里可以修改

xTimerGetTimerDaemonTaskHandle // 返回与软件定时器守护进程（或服务）任务关联的任务句柄

xTimerPendFunctionCall // 允许在FreeRTOS的任务或定时器回调函数中，延迟执行一个函数调用到空闲任务（idle task）的执行环境

xTimerPendFunctionCallFromISR // 可从中断服务例程调用的 xTimerPendFunctionCall() 的版本

xTimerGetPeriod // 返回软件计时器的周期，周期在创建计时器时配置，也可以通过xTimerChangePeriod修改

xTimerGetExpiryTime // 用于获取一个定时器的到期时间，允许用户查询定时器的下一次触发时间

xTimerGetReloadMode // 设置定时器的模式，有自动重载定时器和一次性定时器

```

## 12.事件组或标志 Event Groups or Flags
事件组（Event Groups）是一个用于不同任务之间同步的机制。在许多使用场景中，使用任务通知notify要比使用事件组的速度更快，内存效率更高。
必须将 RTOS 源文件 FreeRTOS/source/event_groups.c 包含在构建中
```
xEventGroupCreate // 创建一个新的 RTOS 事件组，并返回可以引用新创建的事件组的句柄

xEventGroupCreateStatic // 创建一个新的 RTOS 事件组，并返回一个句柄，用户提供空间

xEventGroupWaitBits // 用来等待一个或多个事件位被设置

xEventGroupSetBits // 在任务上下文中设置事件组（Event Group）中一个或多个位‌，将指定事件组中的某些位（bit）置为 1，表示对应事件已发生，用来实现任务间的同步与通信

xEventGroupSetBitsFromISR // 可以在中断服务程序（ISR）中直接使用的xEventGroupSetBits的版本

xEventGroupClearBits // 用于清除事件组中的一位或多个位

xEventGroupClearBitsFromISR // 可以在中断服务程序（ISR）中直接使用的xEventGroupClearBits的版本

xEventGroupGetBits // 返回 RTOS 事件组中事件位（事件标志）的当前值，不能从中断使用此函数

xEventGroupGetBitsFromISR // 可以在中断服务程序（ISR）中直接使用的xEventGroupGetBits的版本

xEventGroupGetStaticBuffer // 检索指向静态创建的事件组数据结构体缓冲区的指针

xEventGroupSync // 用于等待一个或多个位在事件组中被设置，并且同时将这些位清除，对于确保只有在特定事件发生后才开始执行某些任务特别有用

vEventGroupDelete // 删除时间组，在被删除的事件组上阻塞的任务将被取消阻塞

```

