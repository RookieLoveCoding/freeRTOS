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

# freeRTOS API引用
## 1.创建任务
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

## 2.任务控制
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

## 3.任务实用程序
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

## 4.RTOS内核控制
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

## 5.RTOS任务通知
### xTaskNotifyGive 和 xTaskNotifyGiveIndexed
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