---
title: "切換Xcode版本"
categories:
  - iOS
tags:
  - Xcode
  - iOS
---
如果有二個Xcode、正式版和測試版<br/>
正式版路徑：`/Applications/Xcode.app/Contents/Developer`<br/>
測試版路徑：`/Users/David/Downloads/Xcode.app/Contents/Developer`<br/><br/>
要從正式版切到正式版的話
```
sudo xcode-select --switch /Users/David/Downloads/Xcode.app/Contents/Developer
```
查看現在在用的Xcode路徑：
```
xcode-select --print-path
```