---
title: "map vs compactMap vs flatmap"
categories:
  - iOS
tags:
  - 高階函數
  - array
---

### map vs compactMap
compactMap的array不會包含nil值、但map會
```
let numbersWithNil = [5, 10, nil, 20]
let doubleNumbers = numbersWithNil.map {$0 * 2}
//會有error
let doubleNumbers = numbersWithNil.map {$0 != nil ? $0! * 2 : nil}
print("doubleNumbers:\(doubleNumbers)")
//才會顯示
//[Optional(10), Optional(20), nil, Optional(40)]
//但如果
let newDoubleNumbers = numbersWithNil.compactMap {$0 != nil ? $0! * 2 : nil}
print("newDoubleNumbers:\(newDoubleNumbers)")
//就可以直接顯示不含nil的值
//[5, 10, 20]
```

### map vs flatMap
flatMap可以把2維陣列變成1維陣列、然後如果2維陣列有nil的話1維陣列也會包含
```
let arrays = [[1, 2, 3], [4, 5, 6]]
//如果是用傳統的for-in
var allArrays: [Int] = []
for oneArray in arrays {
    allArrays += oneArray
}
print("allArrays:\(allArrays)")
//會顯示： allArrays:[1, 2, 3, 4, 5, 6]
//但如果用flatMap的話一行就可以解決
let newArrays = arrays.flatMap{ $0 }
print("newArrays:\(newArrays)")
//會顯示： newArrays: [1, 2, 3, 4, 5, 6]
// 另外flatMap會顯示nil所以
let nilArrays = [[1, 2, 3, nil], [4, nil, 5, 6]]
let newNilArrays = nilArrays.flatMap{ $0 }
print("newNilArrays:\(newNilArrays)")
//會顯示：newNilArrays:[Optional(1), Optional(2), Optional(3), nil, Optional(4), nil, Optional(5), Optional(6)]
```

