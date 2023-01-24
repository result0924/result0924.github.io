---
title: "CoreData Note"
categories:
  - iOS
tags:
  - core data
---

### Core data migration

1. 判斷要不要migration

    ```text
    * 完全不要migration? 直接清掉資料庫重建？
    * 版本不一樣才migration? 從那一版開始migration
    ```

2. 先check WAL是否已經備份完成

    ```text
    Since iOS 7, Core Data has used the [Write-Ahead Logging (WAL)](https://www.sqlite.org/wal.html) option on SQLite stores 
    to provide the ability to recover from crashes by allowing changes to be rolled back until the database is stable. 
    If you have ever had to perform a rollback before, the WAL approach may work a little differently from what you are expecting. 
    Rather than directly writing changes to the sqlite file and having a pre-write copy of the changes to rollback to, 
    in WAL mode the changes are first written to the sqlite-wal file and at some future date those changes are transferred to the sqlite file. 
    The sqlite-wal file is in effect an up-to-date copy of some of the data stored in the main sqlite file.

    The sqlite-wal and sqlite files store their data using the same structure to allow data to be transferred easily between them. 
    However, this shared structure causes issues during migration as Core Data only migrates the data stored in the sqlite file 
    to the new structure, leaving the data in the sqlite-wal file in the old structure. 
    The resulting mismatch in structure will lead to a crash when Core Data attempts to update/use data stored in the sqlite-wal file 😞 . 
    To avoid this crash, we need to force any data in the sqlite-wal file into the sqlite file 
    before we perform a migration - a process known as checkpointing:
    ```

3. 從mom或metadata的identifier去取得目前的app version

    - What is mom

    ```text
    When Xcode compiles your app into its app bundle, 
    it will also compile your data models. 
    The app bundle will have at its root a .momd folder that contains .mom files. 
    MOM or Managed Object Model files are the compiled versions of .xcdatamodel files. 
    You'll have a .mom for each data model version. 
    ```

    - How to find persistent store model version?

    ```text
    These properties will allow you to access the current store URL and model. 
    As it turns out, there is no method in the CoreData API to ask a store for its model version.
    Instead, the easiest solution is brute force. 
    Since you've already created helper methods to check if a store is compatible with a particular model, 
    you'll simply need to iterate through all the available models until you find one that works with the store.
    ```

    ```swift
    lazy var currentModel: NSManagedObjectModel =  {   
      // Let core data tell us which model is the current model   
      let modelURL = NSBundle.mainBundle().URLForResource(self.modelName, withExtension:"momd")   
      let model = NSManagedObjectModel(contentsOfURL: modelURL!)   
      return model 
    }()
    ```

    - How to find current model version

    ```swift
    // 可以用NSManagedObjectModel的isConfiguration來比較差異
    /* 
    Compares the version information in the store metadata with the entity version of a given configuration. 
    Returns NO if there are differences between the version information.  
    (For information on specific differences, developers should utilize the entityVersionHashesByName method, and perform a comparison.)
    */
    @available(iOS 3.0, *)
    open func isConfiguration(withName configuration: String?, compatibleWithStoreMetadata metadata: [String : Any]) -> Bool

    ```

4. Debug

    ```text
    A useful launch argument when testing migrations is -com.apple.CoreData.MigrationDebug. When set to 1, 
    you will receive information in the console about exceptional cases as it migrates data.
    If you’re used to SQL but new to Core Data, set -com.apple.CoreData.SQLDebug to 1 to see actual SQL commands.
    ```

5. Some note

    ```text
    Both Lightweight and Standard migration techniques can be achieved automatically or manually. 
    By default with NSPersistentContainer Core Data will attempt to perform Standard 
    and then fall back to Lightweight if it can't find the needed mapping model. 
    ```

### Core data

The Core Data stack 處理與外部數據存儲的所有交互，以便您的應用程序可以專注於其業務邏輯

The stack consists of four primary objects:

- managed object context ([NSManagedObjectContext](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext))

```text
The managed object context (NSManagedObjectContext) is the object that your application will interact with the most, and therefore it is the one that is exposed to the rest of your application. 
```

- persistent store coordinator ([NSPersistentStoreCoordinator](https://developer.apple.com/documentation/coredata/nspersistentstorecoordinator))

```text
The NSPersistentStoreCoordinator sits in the middle of the Core Data stack.
The coordinator is responsible for realizing instances of entities that are defined inside of the model. 
```

- managed object model ([NSManagedObjectModel](https://developer.apple.com/documentation/coredata/nsmanagedobjectmodel))

```
The NSManagedObjectModel instance describes the data that is going to be accessed by the Core Data stack.
```

- persistent container ([NSPersistentContainer](https://developer.apple.com/documentation/coredata/nspersistentcontainer)).

```
handles the creation of the Core Data stack and offers access to the NSManagedObjectContext as well as a number of convenience methods
```

### Refer

- [progressive-core-data-migration](https://williamboles.com/progressive-core-data-migration/)
- [InitializingtheCoreDataStack](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/InitializingtheCoreDataStack.html)
- [objc.io: core-data-migration](https://www.objc.io/issues/4-core-data/core-data-migration/)
- [Master in core data](https://ali-akhtar.medium.com/mastering-in-coredata-part-1-introduction-7c5d667bfabd)

### Core Data Batch Insert

- [nsbatchinsertrequest](https://swiftstudent.com/2020-03-29-nsbatchinsertrequest/)
- [Apple example](https://developer.apple.com/documentation/coredata/loading_and_displaying_a_large_data_feed)

### Background task

- [What's the difference between performBackgroundTask and newBackgroundContext()?](https://stackoverflow.com/questions/53633366/coredata-whats-the-difference-between-performbackgroundtask-and-newbackgroundc)
- [Core data performance](https://www.avanderlee.com/swift/core-data-performance/)

### Debug

- [Core data debug](https://www.avanderlee.com/debugging/core-data-debugging-xcode/)
