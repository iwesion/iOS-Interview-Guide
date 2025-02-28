# 线程通信

## 基本概念

线程通信是指线程之间进行数据传递和同步的机制。在 iOS 中，有多种方式可以实现线程间的通信，包括直接通信和间接通信。

## 通信方式

### 1. GCD
```objc
// 主线程更新 UI
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 后台处理
    NSString *result = [self processData];
    
    dispatch_async(dispatch_get_main_queue(), ^{
        // 主线程更新 UI
        self.label.text = result;
    });
});
```

### 2. NSOperation
```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    // 后台处理
    NSString *result = [self processData];
    
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        // 主线程更新 UI
        self.label.text = result;
    }];
}];
[queue addOperation:operation];
```

### 3. performSelector
```objc
// 在主线程执行
[self performSelectorOnMainThread:@selector(updateUI:) 
                     withObject:data 
                  waitUntilDone:NO];

// 在后台线程执行
[self performSelectorInBackground:@selector(processData) 
                     withObject:nil];
```

## 通信机制

### 1. 通知中心
```objc
// 发送通知
[[NSNotificationCenter defaultCenter] postNotificationName:@"DataUpdated" 
                                                  object:nil 
                                                userInfo:@{@"data": data}];

// 接收通知
[[NSNotificationCenter defaultCenter] addObserver:self 
                                       selector:@selector(handleDataUpdate:) 
                                           name:@"DataUpdated" 
                                         object:nil];
```

### 2. KVO
```objc
// 添加观察者
[object addObserver:self 
        forKeyPath:@"property" 
           options:NSKeyValueObservingOptionNew 
           context:nil];

// 处理变化
- (void)observeValueForKeyPath:(NSString *)keyPath 
                    ofObject:(id)object 
                      change:(NSDictionary *)change 
                     context:(void *)context {
    if ([keyPath isEqualToString:@"property"]) {
        // 处理属性变化
    }
}
```

### 3. 代理模式
```objc
@protocol DataDelegate <NSObject>
- (void)didUpdateData:(NSString *)data;
@end

@interface DataProcessor : NSObject
@property (nonatomic, weak) id<DataDelegate> delegate;
@end

@implementation DataProcessor
- (void)processData {
    // 处理数据
    [self.delegate didUpdateData:result];
}
@end
```

## 实践案例

### 1. 网络请求
```objc
@interface NetworkManager : NSObject

- (void)fetchDataWithCompletion:(void(^)(id result))completion {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 网络请求
        NSData *data = [self downloadData];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            if (completion) {
                completion(data);
            }
        });
    });
}

@end
```

### 2. 图片加载
```objc
@interface ImageLoader : NSObject

- (void)loadImageWithURL:(NSURL *)url completion:(void(^)(UIImage *image))completion {
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    [queue addOperationWithBlock:^{
        NSData *data = [NSData dataWithContentsOfURL:url];
        UIImage *image = [UIImage imageWithData:data];
        
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            if (completion) {
                completion(image);
            }
        }];
    }];
}

@end
```

### 3. 数据同步
```objc
@interface DataSynchronizer : NSObject

- (void)syncData {
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    dispatch_group_enter(group);
    [self fetchDataFromServer:^(id result) {
        dispatch_group_leave(group);
    }];
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 所有数据同步完成
        [self updateUI];
    });
}

@end
```

## 性能优化

### 1. 选择合适的通信方式
```objc
// 简单通信用 GCD
dispatch_async(queue, ^{});

// 复杂任务用 NSOperation
NSOperation *operation = [NSOperation new];
```

### 2. 避免过度通信
```objc
// 批量处理
NSMutableArray *updates = [NSMutableArray array];
// 收集更新
dispatch_async(dispatch_get_main_queue(), ^{
    // 一次性更新 UI
});
```

## 注意事项

1. **线程安全**
   - 注意数据竞争
   - 使用同步机制
   - 避免死锁

2. **内存管理**
   - 注意循环引用
   - 正确移除观察者
   - 处理异步操作的内存问题

3. **性能考虑**
   - 选择合适的通信方式
   - 避免过度通信
   - 注意通信开销

## 面试要点

1. iOS 中有哪些线程通信方式？
- GCD: dispatch_async, dispatch_sync 等
- NSOperation: 操作队列和依赖关系
- performSelector: 在指定线程执行方法
- NSNotification: 观察者模式的通知机制
- KVO: 键值观察
- delegate: 代理模式
- block: 闭包回调

2. 如何在后台线程进行网络请求并更新 UI？
- 使用 GCD:
  ```objc
  dispatch_async(global_queue, ^{
      // 网络请求
      dispatch_async(main_queue, ^{
          // UI 更新
      });
  });
  ```
- 使用 NSOperation:
  ```objc
  [queue addOperationWithBlock:^{
      // 网络请求
      [[NSOperationQueue mainQueue] addOperationWithBlock:^{
          // UI 更新
      }];
  }];
  ```

3. NSNotification 和 KVO 的区别？
- NSNotification:
  - 一对多广播机制
  - 可传递任意数据
  - 松耦合
  - 需要手动发送通知
- KVO:
  - 一对一观察机制
  - 只能观察对象属性
  - 自动触发
  - 系统实现属性变化通知

4. 如何处理异步操作的回调？
- Block 回调
- Delegate 回调
- 通知机制
- Promise/Future 模式
- Completion Handler
- 注意避免循环引用
- 考虑线程安全

5. 线程通信中如何避免死锁？
- 避免循环等待
- 使用 dispatch_async 代替 dispatch_sync
- 避免在主线程同步等待
- 使用 dispatch_group
- 合理设计依赖关系
- 使用信号量控制并发

6. 不同通信方式的优缺点？
- GCD:
  - 优: 简单高效,性能好
  - 缺: 功能相对简单
- NSOperation:
  - 优: 功能丰富,可控性强
  - 缺: 相对复杂,开销大
- Notification:
  - 优: 解耦,广播机制
  - 缺: 维护成本高
- KVO:
  - 优: 自动触发,直观
  - 缺: 耦合度高

7. 如何选择合适的通信方式？
- 简单任务用 GCD
- 复杂任务用 NSOperation
- 一对多通知用 Notification
- 属性监听用 KVO
- 对象间通信用 delegate
- 注意性能和维护成本
- 考虑线程安全问题

## 相关资源

- [Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html)
- [Notification Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Notifications/Introduction/introNotifications.html)
- [Key-Value Observing Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html) 