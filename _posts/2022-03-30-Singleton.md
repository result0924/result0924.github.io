---
title: "Singleton"
categories:
  - 設計模式
tags:
  - Singleton
last_modified_at: 2022-03-30T10:28:00
---

### 什麼是Singleton?
- 確保class只有一個instance、並且確保只有single point可以去存取
- 確保沒有其它instance可以依靠攔截請求來創建新的物件
- 提供訪問唯一實例的方法
- 確保class不能從外部多次實體化
- 不禁止開發人員創建該類的新實例、但約束取決於開發人員的選擇或紀錄、僅將其實例化一次
- Apple's `URLSeesion.shared`和`UserDefaults.standard`都是Singleton

### 何時該用Singleton?
- 系統隨時都有可能去存取功能、ex. UserDefault
- Singleton應該很少見、並且要與系統有一對一的關系

### 何時不該用Singleton?
- 只被創建一次的對像
- View, ViewModel, Presenter都是不好的Singleton

### Refer
- [Careful With “Singleton” Lookalikes (WAY TOO COMMON)](https://www.essentialdeveloper.com/articles/careful-with-singleton-lookalikes-way-too-common)