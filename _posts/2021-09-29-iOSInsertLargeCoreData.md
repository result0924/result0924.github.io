---
title: "iOS Insert Large Core Data"
categories:
  - iOS
tags:
  - core data
  - iOS
---

如果有大量的資料要匯入CoreData<br/>
iOS13以後有個function<br/>
NSBatchInsertRequest可以使用<br/>

Refer: <cite><a href="https://developer.apple.com/documentation/coredata/loading_and_displaying_a_large_data_feed">Apple's Document</a></cite>

iOS 13以前的話就每次save完要call reset()
```scala
Apps targeted to run on a system earlier than iOS 13 or macOS 10.15 
need to avoid memory footprint growing by processing the objects 
in batches and calling reset() to reset the context after each batch.
```