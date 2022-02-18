---
title: "Xcode M1 Setting"
categories:
  - iOS
tags:
  - Xcode
  - M1
---

### Homebrew

- [M1 安裝](https://stackoverflow.com/a/64997047)

### Carthage

- brew install carthage
- must use `--use-xcframeworks` ex. `./wcarthage bootstrap --platform iOS --no-use-binaries --use-ssh --use-xcframeworks`
- [command PhaseScriptExecution failed with a nonzero exit code](https://stackoverflow.com/a/70860322)

```
rm -rf ~/Library/Developer/Xcode/DerivedData
rm -rf ~/Library/Caches/org.carthage.CarthageKit/DerivedData
./wcarthage bootstrap --platform iOS --no-use-binaries --use-ssh --use-xcframeworks
```

### Cocoapods

 - [[iOS 教學] 在 Apple M1 機器上面正確編譯 iOS 專案](https://blog.jks.coffee/compile-cocoapods-project-with-apple-m1/)

### swiftLint

- [SwiftLint working from command line but says not installed in Xcode](https://github.com/realm/SwiftLint/issues/2992#issuecomment-771001634)

```
if test -d "/opt/homebrew/bin/"; then
    PATH="/opt/homebrew/bin/:${PATH}"
fi

export PATH

if which swiftlint >/dev/null; then
    swiftlint
else
    echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```
    
### Error

- [ls command not found](https://stackoverflow.com/questions/57972341/how-to-install-and-use-gnu-ls-on-macos)