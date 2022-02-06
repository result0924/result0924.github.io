---
title: "Unknow enum case"
categories:
  - iOS
tags:
  - enum
---

### Unknow enum case
使用情境：原本後端Api定義了二種情境 <br/>
ex. success / fail <br/>
所以Codebale的enum就只需要定義success和fail <br/>
但如果下一個版本突然加了一個pandding那enum就會出現問題 <br/>
導致這版本的app無法使用必須強制升級、或後端得根據版本給值或使用新的api <br/>
此篇作者用三個方式去防止這情況發生、並解釋何時該用什麼 <br/>

### Refer
- [Handling unknown enum cases](https://5sw.de/2022/01/unknown-enum-cases/)