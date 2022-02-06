---
title: "Unknow enum case"
categories:
  - iOS
tags:
  - GCD
---

### Building Responsive and Efficient Apps with GCD
- GCD：多線程處理
    - 在主線程處理UI、可能需要讀取資料、如果在主線程讀取資料再等到回應可能需要等待、此時如果用GCD在背景處理就不用等待。

- Quality of Service Classes(Complex resource controls)
    - CPU scheduling priority (我們要運行那些線程以什麼順序進行)
    - I/O priority (當系統中有多個I/O時、應該以什麼順序執行)
    - Timer coalescing (計時器聚合技術是一個省電的功能)
    - CPU throughput vs efficiency (我們按照吞吐量還是以效率導效模式運行CPU)
    - More... (我們想要獲得最佳的性能還是想要、以最節能的方式執行代碼)
    - Configuration values tuned for each platform/device (在理想情況下為每個平台或設備優化)

- QoS有四個質量類別
    - User Interacitve (Main thread, animation, Is this work actively involved in updating the UI?)
    - 這是主線程、主要用放交互代碼是保持動畫
    - User Initated (Immediate result)
    - 是指加載由用戶完成的動作結果、比如滑動畫面為下一個畫面加載數據時
    - Utility (Logn-running tasks)
    - 是指用戶本來可能已經開始進行、或已經自動啟用的任務、這些任務長時間運行但是並不阻礙用戶、繼續使用你們的程式
    - 例如在背景下載資料
    - Background (Not user viible)
    - 用戶根本沒意識到這部分

- GCD Design Patterns with QoS
    - GCD and QOS (Fundamentals)
    - QoS can be specified on Blocks and on queues, dispatch_async() automatically propageaters Qos Some priority inversions are resolved automatically

- Qos Propagation(Inferred Qos)
    - Qos captured at the time of Block Submission
        - User Interactive translated to User Initiated
    - User if destination does not have Qos specified
        - Does not lower QoS

- Block Qos (^{...})
    - Block created with explicit Qos attribute
        - When work of another class is generated
    - Captured at Block object creation
        - DISPATCH_BLOCK_ASSIGN_CURRENT
        - Store a callback Block for later submission

- Asynchronous Priority Inversion
    - High QoS Block submitted to serial queue
        - Queue already contains Blocks with lower Qos
    - QoS is raised until high QoS Block is reached
        - Invisible to Blocks themselves

- Queue QoS(recap)
    - Queues that are single purpose
        - Detached Blocks may also be appropriate
    - Ignore QoS asynchronous Blocks
        - Exceptional cases can enforce BLock QoS

- Synchronous Priority Inversion
    - High QoS thread wating on lower QoS work
    - Qos of waited on work is raised for
        - dispatch_sync() and dispatch_block_wait() of Blocks on serial queues
        - pthread_mutex_lock()

- Run Loop Versus Queue
```
    dispatch_async(q, ^{
        [self performSelector:@selector(thin) withObject:nil afterDelay:1];
    });
    上面的程式塊完成後、線程可能消失、這是我們臨時線程池裡的線程、
    他們沒有任何可確保的有效期、我們可能破壞這個線程
    當然即使線程保留了下來、也沒人在實際地運行那個運行循環
    那個計時器將永遠不會被觸發
```
    - Run Loop
        - Bound to thread(運行循環受限於某一特定線程)
        - Gets delegate method callbacks(一般看到API通過委托方式被回調)
        - Autorelease pool pops after each iteration
        - (他們擁有在整個運行循環源中的每次迭代後出現的自動釋放池)
        - Can be used reentrantly(可以重新進入使用)
    - Serial Queue
        - Uses ephemeral threads(使用來自GCD線程池的臨時線程)
        - BLock callbacks(一般將程序塊作為它們的回調)
        - Autorelease pool pos when thread idle
        - (僅在線程完全空閒時、串行隊列的自動釋放池才會出現、如果應用持續忙碌中的話、自動釋放池將絕不會出現)
        - will deadlock if used reentrantly(不是一個重入鎖、或遞歸鎖結構)
        - 你需要確保當你設計你應用的隊列時、不要使自已陷入需重入地使用它們的情況
    - The Main Threads's Run Loopis also exposed as the Main Queue

- Timer APIs
    - RunLoop
        - [NSObject performSelector:withObject:afterDelay]
        - [NStimer scheduleTimerWithTimeInterval:]
    - Queue
        - dispatch_after()
        - dispatch_source_set_timer()

- Waiting
```
A thread waits(blocks) when it needs to wait for a resource such as I/O or locks
When a thread waits, GCD amy spin up a new thread to ensure one thread per core
```

- Thread Explosion Causing Deadlock
```
實際案例：
我們有主線程、並且主線程有大量的工作需要去做、它會異步調度大批程序塊
到某一並發隊列、我們為這些程序塊提出線程、這些程序塊、立即轉向同步調度回主線程
這時我們已經提出了、我們需要所有的線程、在這裡的簡單例子中是四個
我們達到了線程限制、我們將不能再為線程池提出任何更多的線程
那麼好的、我們需要主線程再次變成可用的
以使那些程序塊可以獲取它、繼續運行至完成
現在它使我們重新回到限制下、僅在某些情況下、這可能發生
但讓我們試想一下、主線程進入異步調度成為某一串行信號
到目前為止一切順利，那個程序塊仍不會開始執行、
因為沒有可利用的額外的主線程、來執行那程序塊
它會呆在那裡、等待著某一線程返回、然後我們可將其再度利用
但是之後我們的主線程決定同步調度成相同的串行隊列
問題是對於串行隊列、沒有可用的線程、主調用將永遠阻塞
這就是經典的死鎖情況(所有線程池中的線程都在等待資源)
```
<blockquote class="imgur-embed-pub" lang="en" data-id="a/ilAQPOx" data-context="false" ><a href="//imgur.com/a/ilAQPOx"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

- Avoiding Thread Explosion
    - Always good advice: use asynchronous APIs, espically for I/O
    - (在任何可能的情況下、儘可能利用異步API、尤其對於I/O、如果這樣做你可以避免程序塊等待、這樣你就不用提出更多的線程)
    - Use serial queues
    ```
    對這類的情況、你也可以用串行隊列、如果調度所有此類工作到串行隊列上、我們就不會遇到線程激增、我們每次將僅執行一個程序塊
    我並不是在告訴你們串行化你們全部的應用程序 但是當你建立你的應用程序、並創建不同的隊列、在你的應用程序中、處理不同的模塊時
    除非你知道對特別的模塊你需要、平行運行某些東西、取而代之地為達到你的性能目標、請考慮從串行隊列開始
    在串行隊列上同時運行你應用程序中的各部分、從中你可獲得很多性能、之後你可以給出你應用程序的輪廓
    看什麼地方需要額外的平行運行的性能、並對這些地方進行特別設計、以避免線程激增
    ```
    - Use NSOperationQueues with concurrency limits
        - NSOperationQueue.maxConcurrentOperatorationCount
    - (或使用有並發限制的NSOperationQueues)
    - Don't generate unlimited work...
    - (最後不要產生無限的工作、如果你可以按照你需要的程序塊數量來約束你的工作、就會避免線程激增)
    - 這是出現問題的代碼(Mixing sync and async)
    ```
    // fast, just a lock
    dispatch_sync(q, ^{...});
    如果我只是對某隊列進行同步調度會非常快、大致會得到鎖
    // fast, jus a atomic enqueue
    dispatch_async(q, ^{...});
    如果我進行異步調度也會非常的快、大體上是原子入隊
    // slow, has to wait for a thread to complete above Block
    dispatch_sync(q, ^{...});
    如果我有一個隊列且僅用二個中的某個、其性能與這些基本是非常相近的

    Be super careful about mixing these from the main thread
    如果我將這些混合異步到某隊列、並做同步調度、
    同步調度就必須等待一線程被建立、並執行那個程序塊、然後程序塊才會被完成

    // DANGEROUS - may cause thread explosion and deadlocks
    for (int i=0; i < 000; i++>) {
        dispatch_async(q, ^{...});
    }
    dispatch_barrier_sync(q, ^{});

    // GOOD - GCD will manage parallelism
    dispatch_apply(999, q, ^(size_t i){...};)

    // define CONCURRENT_TASKS 4
    sema = dispatch_semaphore_create(CONCURRENT_TASKS);
    for (int i = 0; i < 999; i++) {
        dispatch_async(q, ^{
            // do work
            dispatch_semphore_signal(sema);
        })
        dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
    }
    ```
- Summary
    - An efficient and responsive application must be able to adopt to diverse environments
    - QoS classes enable the system to manage resources appropriately
    - Integrate QoS into your application and existing use of GCD
    - Consider your app's use of GCD and avoid thread explosion
```

### Refer
[WWDC 2015 Building Responsive and Efficient Apps with GCD
](https://developer.apple.com/videos/play/wwdc2015/718/)