---
title: "Test Thread Safe"
categories:
  - iOS
tags:
  - unit test
  - TDD
  - thread
---

### Thread safe data structure
1. Updates could arrive in a different order (an issue ony if we value ordering)
2. We can miss some update
3. The app can crash
When a data struture is implement in a way that prevents these issues, we call it a thread-safe data structure

### TDD follow three steps
1. We create a failing test.
2. We write the minimum code to make it pass.
3. Eventually, we refactor the code to make it more readable and maintainable.

### Refer

[Test thread safe](https://betterprogramming.pub/how-to-test-if-your-swift-classes-are-thread-safe-before-and-after-actors-efc743f27da4)
[Test asynchronous test and expectations](https://developer.apple.com/documentation/xctest/asynchronous_tests_and_expectations/testing_asynchronous_operations_with_expectations)
[非同步測試](https://medium.com/@ji3g4kami/unit-test-%E6%95%99%E5%AD%B8-%E9%9D%9E%E5%90%8C%E6%AD%A5%E6%B8%AC%E8%A9%A6-api-a5d7f2777167)
