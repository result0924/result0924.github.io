---
title: "cocapods code sign error"
categories:
  - ios
tags:
  - cocoapods
---

### code sign error when use Xcode 14

It looks like in Xcode 13 CODE_SIGNING_ALLOWED was set to NO by default for resource bundles. In Xcode 14 they changed this to default to YES, which is what is causing the problems.

２種解決方法：

  1. 
  `config.build_settings['CODE_SIGNING_ALLOWED'] = 'NO'`

  2. 
```
  config.build_settings["DEVELOPMENT_TEAM"] = "YOUR TEAM ID"
  config.build_settings["CODE_SIGN_IDENTITY"] = "Apple Distribution";
  config.build_settings["CODE_SIGN_STYLE"] = "Manual";
```

soluction:[https://github.com/CocoaPods/CocoaPods/issues/11402#issuecomment-1323767340](https://github.com/CocoaPods/CocoaPods/issues/11402#issuecomment-1323767340)

ps. 第１種方法disable code sign 可能會審核不過
refer:[https://github.com/CocoaPods/CocoaPods/issues/11402#issuecomment-1287590272](https://github.com/CocoaPods/CocoaPods/issues/11402#issuecomment-1287590272)