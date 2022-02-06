---
title: "Memrory Check"
categories:
  - iOS
tags:
  - memory
  - intrument
  - WWDC
---

## Detect and diagnose memory issues

### Issues to look for

- Leaks
  - (在進程分配一個對象、但沒有釋放它的情況下、卻失去所用相關引用時、就會產生泄漏)
  - Allocated objects to which there are no active references
  - Be cautionus when using unsafe types in swift
  - Wath out for retain cycle, avoid circular strongrefs
  - (如果循環引用真的很必要的話、那就考慮用一個弱引用吧、因為弱引用不會阻檔對象被釋放)

- Heap size issues
  - Heap allocation regressions
    - Heap stores dynamically allocated objects
    - (堆就是你的進程地址空間區、存放動態分配對象的地方)
    - Allocating more objects on heap increases footprint
    - (堆分配回歸、就是由于進程分配比以前還多的對象在堆上、所造成的內存占用)
    - Remove unused allocations
    - (為了減少內存占用、要移除沒有使用的分配)
    - Shrink overly large allocations
    - (並縮小非必要的巨大分配)
    - Deallocated memory you are finished with
    - (釋放你沒有在使用的內存)
    - Wait to allocated memory until you nedd it
    - (等到你需要的時候再分配內存)
    - 這就會減少你的程序高峰占用值、讓它不太會被終止運行

  - Fragmentation(碎片化)
    - Pages are smallest indivisible unit of memory for your process
    - (一個頁是固定大小、獨立的內存單元、是系統給予你的進程的)
    - Writing anything to a page makes the whole page dirty
    - (也由於頁是獨立的、當你的進程寫入任何一部分的頁、整頁就變成髒頁、也會占到你的進程、就算大都是未使用的也是一樣)
    - Unused stretches of memory on dirty pages that are too small to use for new allocations
    - (碎片化會在進程有了髒頁、卻沒有100%利用它時發生)
    - Allocate objects with similar lifetimes close to each other
    - (減少碎片化的最佳方法是讓有相似生命週期的對象、在內存裡盡可能連續靠近彼此)
    - (這會幫助確保全部的對象都能同時被釋放、讓進程中比較巨大的連續性內存、能夠被利用在未來的分配上)
    - Aim for 25% fragmentation or less
    - (根據經驗法則把目標放在大約25%或更少的碎片方即可)
    - Use autorelease pools
    - (一種減少碎片化的方法是用自動釋放池、自動釋放池會讓系統知道、只要對象不再作用、就要釋放其中所有的對象)
    - (這會幫助確保所有在自動釋放池內的對象、有著相似的生命週期)
    - Pay extra attention to long running processes
    - (雖然所有的進程、都有可能碰到碎片化問題、但特別可能發生在長時間運行的進程中)
    - (有可能是另為很多分配跟釋放、導致地址空間被分成碎片)

### Wrap up

  - Eliminate memory leak
  - (力求程式、無內存洩漏、如果你在處理的是unsafe類別、記得要釋放每個你分配的東西)
  - (並注意你的程序代碼裡是否有循環引用)
  - Reduce heap allocations where possible
  - (想辦法減少你的堆內存分配、無論是縮小它們、或減少運行它們的時間、或是移除不必要的分配)
  - Reduce fragmentation with good allocation practices
  - (記得要將碎片化放在心裡、將有相似生命週期的對象分配到一起、之後才能創造很好的、較大空閒內存空間)

## WWDC19 Getting Started with Instruments

- Instrument是在Xcode工具集中的一個強大的性能分析和測試工具
- Instrument會從插入在App進程和操作系統的重要內部架構中收進時間序列追蹤數據
- 有時我們會稱Instrument收集到的數據為`Treat`

### Time Profiler

- 使用操作系統提供的內部架構、以固定的時間間隔、來收集相關線程的所有呼叫棧
- 分析在某個時間發生的問題的絕佳工具
- `Spinning`在Instrument中代表你的主線程卡住了
- Tips
  - Time Profile shows how your app is spending time
  - Check main thread when responsiveness issues occur
  - Profile release builds
  - Profile with difficult workloads and older devices
- High CPU effect
  - 榨乾電池容量(Drain the battery)
  - 提高設備溫度(Heat up the device)
  - 風扇加速運轉(Spin up fans)

### Points

- 從你App中重要的區域收集數據、讓你能用多種API對其加以強調
  - ex. Signposts API

- Using Signposts
  - Time Profiler它是通過在一個固定時間間隔內觀測你的App中的所有線程、並構建出時間和調用棧的相關分析、但相關分析並不能取代精準評估
  - 為了區分這幾種執行模式、你需要記錄代碼的精確評估、這時就需要使用Signposts
    1. 可能有代碼會在幾個時間點爆發性地執行
    2. 或者它在幾個更長的時間段內持續執行
    3. 在某些特定的Argument調用的某個函數、可能讓CPU持續繁忙

- Features Of Signposts
  - Simpler and more efficient the printing
  - Built-in support for measuring time
  - Traced by Instruments

- Singpost log
  - `mach_absolute_time()`
  - Instrument並不知道如何讀取打印命令、所以需要寫Code來與Instruments進行通訊
  - OSLog category `.pointsOfInsterest`
  - os_signpost (begin/end)

- Concepts
  - Statistical profiles show which code is most commonly executed
  - Exact measurements show how and why code is executed
  - XCTest reliably reproduce workloads for profiling

## WWDC18 iOS Memory Deep Dive

- 接近設備的內存限制時會收到`EXC_RESOURCE_EXCEPTION`的異常
- Xcode的內存調試器是用用`Memgraph`文件格式
- 內存都去那裡了?可以用`heap`它提供了內存對象分配的各種信息

### 查看內存

- Creation -> `malloc_history`
- Reference -> `leaks`
- Size -> `vmmap` or `heap`

- 內存使用與圖像的尺寸有關但與檔案大小無關

### Don't pick the format, let the format pick you

- 停止使用`UIGraphicsBeginImageContextWithOptions`
- 開始使用`UIGraphicsImageRenderer`
- UIImage當sizing和resizing是很花內存的、因此最好用`ImageIO`來取代

### 如果使用大張的圖片、當看不見它時最好把它dealloc

- App lifecycle
  - Listen for `UIapplicationWillEnterForeground` and `UIApplicationDidEnterBackground` notifications
  - Applies to on-screen views
- UIViewController apperance cycle
  - Leverage `viewWillAppear` and `viewDidDisappear`
  - Off-screen view controllers in `UITabBarController` and `UINavigationController`

### Summary

- Memory is a finite and shared resource
- Monitor memory use when running from Xcode
- Let iOS pick your image formats
- Use ImageIO for downsampling images
- Unload large resources that are off-screen
- Use memory graphs to further understand and reduce memory footprint

## Refer

- [檢測和診斷App內存](https://mp.weixin.qq.com/s?__biz=MzI2NTAxMzg2MA==&mid=2247492779&idx=1&sn=a371a10a3bcf58acd5593a1a38d6f1db&scene=21#wechat_redirect)
- [WWDC21 Detect and diagnose memory issues](https://developer.apple.com/videos/play/wwdc2021/10180/)
- [WWDC19 Getting Started with Instruments](https://developer.apple.com/videos/play/wwdc2019/411)
- [WWDC18 iOS Memory Deep Dive](https://developer.apple.com/videos/play/wwdc2018/416)