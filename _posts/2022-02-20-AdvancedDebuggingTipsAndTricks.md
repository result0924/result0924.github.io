---
title: "Advanced Debugging Tips and Tricks"
categories:
  - iOS
tags:
  - Debug
  - WWDC
last_modified_at: 2022-02-14T12:07:00
---
### Advanced Debugging Tips and Tricks

- `expression` 可以更改屬性
```
如果不想每次暫停都必須輸入這個表達示、則可以配置這個斷點來為我做這件事
右鍵點擊斷點-> Edit break point -> 用一些配置來定制斷點的行為
-> 將Action設置為Debugger Command -> 並輸入表達示、即在控制台中使用的命令、
ex. expression something = false
-> 並在automaticall continue after evaluating actions這個option打勾
當程式執行到這裡、該斷點將被觸發、為我們更改屬性、然後自動繼續執行
```
- 使用 break poitn -> action 可以不用重build 程式就看到代碼是否成功修復

- 可以新增 error/exception/symbolic breakpoint 去觀察

```
可以檢查傳入函數的參數、`po $arg1` 表示參數的屬性
(lldb) po $arg1
<UILabel: xxxxxx .... layer - <xxxx>>
(lldb) po $arg2
48983242384
如果你熟悉objc_msgSend應該知道第二個屬性是選擇器(selector)
可是我們並沒有看到它、那是因為LLDB不會自動知道這些參數的類型
因此在某些情況下、我們需要對它進行類型轉換
(lldb) po (SEL)$arg2
"setText:"
第三個參數是傳遞到方法中的第一個參數、換句話說、它是傳遞給setText的字符串
所以這對於在一個框架的匯編幀中、檢查參數來說非常方便
(lldb) po $arg3
0 ft
```
- break point 有個condition的欄位、可以在這裡輸入返回true或false的表達式、只有該表達式為true時、才會觸發斷點
```
可以在Action設置一個 `breakpoint set --one-shot true`
這個一次性斷點是一個臨時斷點、一旦觸發後就會被自動刪除
```
<blockquote class="imgur-embed-pub" lang="en" data-id="a/iUjAPnm" data-context="false" ><a href="//imgur.com/a/iUjAPnm"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>
```
這裡的這個看起來像握柄的圖標、實際上就是一個握柄、握住鼠標就可以移動它。
這樣我就可以在暫停時更改指令指針、我可以把它往下移動一行、然後放手、接著Xcode會顯示一條可怕的消息、
它所說的基本就是、巨大的權力會帶來后大的責任、說實話這是一個危險的功能、
這項操作是有風險的、因為調試器允許你移動指令指針到你想要的任何地方、它並不關心
但它不能保證app的狀態、完好無損。例如這可能會造成內存管理問題、
當你試圖引用尚未初始化的對象、或過早釋放對象時。
跳過一行程式碼的lldb是 `thread jump --by 1` 這告訴調試器為當前線程、跳過一行代碼、然後我們只需要調用表達式
我們可以通過點擊加號按鈕來添加另一個命令操作、
```

- 查看view簡單的層次結構 `po self.view.recurisveDescription()`
```
這行不通、為什麼呢?recursiveDescription僅用於調試目的、它不是公共API的一部分、因此也不會被Swift掃描
所以Swift是一種嚴格的語言、不允許你調用、尚未嚴格定義的函數、然而Objective-C不同、在Objective-C世界中
代碼可以自由放縱的運行、你可以做任何你想做的事、我的意思是它是一種動態語言、所以你可以這樣調用函數。
所以我們要做的事、是告訴調試器、使用Objective-C語法、來評做這個表執式。實現這點的方法是
使用expression命令和選項參數 -l objc
`expression -l objc` 這表示你即將給expression命令、一段Objective-C代碼、即使你處於Swift框架中。
我們再加上 -O 來告訴它我們也想要調試描述、就像用po一樣、再加上--以表示沒有更多選項
該行的不餘部分僅僅是原始表達式輸入
`expression -l objc -O -- [self.view recursiveDescription]`
不幸的是這並不奏效、其原另是這個表達式、將為Objective-C編譯創建一個臨時表達式上下文、
並且它不會繼承Swift框架中的所有變量、僅管如此、還是有辦法的
如果我們把self.view放到反引號中、反引號就像預處理器一樣、
它表示先評做其在當前幀中的內容、並插入結果、然後我們可以評估其餘部分
`expression -l objc -O -- [`self.view` recursiveDescription]`
現在我們可以看到遞迴描述了
```

- lldb也可以用命令表名 `command alias poc expression -l objc -O --`

```
(lldb) poc 0x7fb3ksdfkds(Objective-C才能po 記憶體位址)
<viex ......>
```

- 如果在Swift中要查記憶體中的View的話、必須用po unsafeBitCase

```
(lldb) po unsafeBitCaxse(xxxx, to: view.self)
必須明確知道是那個view
unsfeBitCast函數的一個好處是、它返回一個經過類型轉換的結果
所以我們可以調用它的函數和屬性名稱、例如.frame
(lldb) po unsafeBitCaxse(xxxx, to: view.self).fram

可以 po unsafeBitCase().center.y = 300
再 expression CATransaction.flush()
去即時變更View的屬性
```

### Refer
- [WWDC18 Advanced Debugging with Xcode and LLDB](https://developer.apple.com/videos/play/wwdc2018/412/)
- [WWDC2018：效率提升爆表的Xcode和LLDB調試技巧](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/742328/)
