---
title: "NSOperation"
categories:
  - iOS
tags:
  - NSOperation
  - WWDC
---

### NSOperations
- NSoperationQueue
  - High-level dispatch_queue_t
    - maxConcurrentOperationCount
  - Long-running task
  - Subclassable
  - Cancellation
    - Lifecycle
    - <blockquote class="imgur-embed-pub" lang="en" data-id="a/hTBYOHw" data-context="false" ><a href="//imgur.com/a/hTBYOHw"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>
    - 在任何階段都可以取消
    - <blockquote class="imgur-embed-pub" lang="en" data-id="a/7SxVP8w" data-context="false" ><a href="//imgur.com/a/7SxVP8w"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>
    - var cancelled: Bool { get } (對NSoperation而言只是改變了cancelled這個屬性的布林值)
    - Subclasses dictate meaning
    - Susceptible to race conditions(取消不一定可以成功完成、例如在使用者取消時、事情剛好做完就無法取消)
  - Readiness
    - var ready: Bool { get } (like cancel 只是一個bool)
    - execute when ready (意思是操作已經準備好可以開始了)
  - Dependencies
    - First this, then that(我們要先執行這個然後執行那個)
    - Strict ordering
    - Implemented via readiness
    - Not limited by operation queues
      - 操作B將取決于操作A的成功執行、操作B將等到操作A執行完畢後才開始執行
      ```
      let operationA = ...
      let operationB = ...
      operationB.addDependency(ooperationA)
      ```
    - Operation deadlock (關於dependency常會有操作停頓的問題)
      - 如果只是讓A dependency B那不會有問題、如果無意中讓B也 dependency A那就會lock住那麼這二個操作就無法繼續執行、
      - 因為他們二個都在彼此等待完成
    - Abstraction
      - Operations abstract logic (NSOperation是一個極好的方法、可以用來提取代碼中的邏輯)
      - Simplifies logic changes (簡化邏輯變化、因為我們處理的是單獨的工作)
      - Grand Central Dispatch?(多線程優化技術呢?)
    - Block Operations (It's blocks all the way down)
      - NSOpertion to execute a block
      - Dependencies for blocks
    - Extending Readiness (Adding requirements)
      - Only execute this if...(在允許操作的情況下、進行擴展)
      - Examples
        - Nextwork is reachable
        - Access to user location
        - User is login
    - Composing Operations (某個操作總是會接著另一個操作、所以可以打包在一起)
      - ex. Downloading and parsing
    - Mutual exclusivity(視圖故障)
      - Alerts (ex：關掉一個alert又跳出另一個alert，must one alert at a time)
      - Videos (一次只關看一個視頻)
      - Loading database (確保不會一次讀取多個資料庫)
      - 因此我們想出一個方法用於描述互斥性：這種方法是在某些時候只執行一種特定的操作
    - Sample Code 
      - Earthquakes and Operations
        - NSOperation subclass
        - Add `conoditions`
        - Add `observers`
        - Example operation
          - Groups
          - URLSessionTask
          - Location
          - Delay
        - Generates dependencies
        - Defines mutual exclusivity
        - Checks for satisfied conditions
        - Example conditions
          - MutuallyExclusive<T>(這條件表示一個操作、與相同類的其它操作是互相排斥的)
          - Reachability
          - Permissions
        - OperationObserver(Notified about significant events)
          - Operation start
          - Operation end
          - Operation generation
          - Example observers
            - Timeouts
            - Background tasks
            - Network activity indicator
### Summary
  - Operations abstract logic (使用操作可以對應用進行邏輯抽象處理、通過在操作中加入你的邏輯、而且之後很容易進行修改)
  - Dependencies clarify intent (使用相關性可以表達應用之間的關系、這樣可以很容易確保特定行為之間的關系)
  - Describe complex behaviors (操作讓你可以描述複雜的行為、比如互斥性或整合)
  - Enables powerful patterns

### Refer
- [WWDC15 Advanced NSOperations](https://developer.apple.com/videos/play/wwdc2015/226/)
- [Sample Code](https://github.com/miscampbell/Advanced-NSOperations)