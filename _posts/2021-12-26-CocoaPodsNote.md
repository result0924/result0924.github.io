---
title: "cocapods note"
categories:
  - ios
tags:
  - cocoapods
---

### Change cocoaapods version

先解除安裝<br/>
`sudo gem uninstall cocoapods`<br/>
再安裝版本<br/>
`sudo gem install cocoapods -v 1.10.1`

### 查看cocoapods版本

`pod --version`

### 查看安裝過程

`pod install --verbose`


### 只安裝新加的庫

`pod install --no-repo-update`