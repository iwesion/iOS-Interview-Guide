# GCD (Grand Central Dispatch) 详解

## 基本概念

GCD 是 Apple 开发的一个多核编程的解决方案，它是一个在系统层面提供的线程池，用于管理并发任务。GCD 可以自动管理线程的生命周期，并根据系统负载来调整线程数量。

## 核心组件

### 1. Dispatch Queue
```objc
// 串行队列
dispatch_queue_t serialQueue = dispatch_queue_create("com.example.serial", DISPATCH_QUEUE_SERIAL);

// 并行队列
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.example.concurrent", DISPATCH_QUEUE_CONCURRENT);

// 全局队列
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 主队列
dispatch_queue_t mainQueue = dispatch_get_main_queue();
```

### 2. Dispatch Group
```objc
dispatch_group_t group = dispatch_group_create();

dispatch_group_enter(group);
dispatch_group_leave(group);

dispatch_group_async(group, queue, ^{
    // 任务
});

dispatch_group_notify(group, queue, ^{
    // 所有任务完成后的回调
});
```

### 3. Dispatch Semaphore
```objc
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
// 临界区代码
dispatch_semaphore_signal(semaphore);
```

## 常用API

### 1. 异步派发
```objc
// 异步执行
dispatch_async(queue, ^{
    // 后台任务
    dispatch_async(dispatch_get_main_queue(), ^{
        // UI 更新
    });
});

// 同步执行
dispatch_sync(queue, ^{
    // 同步任务
});
```

### 2. 延迟执行
```objc
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), queue, ^{
    // 延迟执行的代码
});
```

### 3. 一次性执行
```objc
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    // 只执行一次的代码
});
```

## 实践案例

### 1. 网络请求
```objc
- (void)fetchData {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 网络请求
        NSData *data = [self downloadData];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            // UI 更新
            [self updateUIWithData:data];
        });
    });
}
```

### 2. 并发下载
```objc
- (void)downloadImages:(NSArray<NSURL *> *)urls completion:(void(^)(NSArray *images))completion {
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    NSMutableArray *images = [NSMutableArray array];
    
    for (NSURL *url in urls) {
        dispatch_group_enter(group);
        dispatch_async(queue, ^{
            NSData *data = [NSData dataWithContentsOfURL:url];
            UIImage *image = [UIImage imageWithData:data];
            if (image) {
                @synchronized (images) {
                    [images addObject:image];
                }
            }
            dispatch_group_leave(group);
        });
    }
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        completion(images);
    });
}
```

### 3. 线程安全的单例
```objc
+ (instancetype)sharedInstance {
    static id instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc] init];
    });
    return instance;
}
```

## 性能优化

### 1. 队列优化
```objc
// 使用适当的队列类型
dispatch_queue_attr_t attr = dispatch_queue_attr_make_with_qos_class(
    DISPATCH_QUEUE_SERIAL, 
    QOS_CLASS_UTILITY, 
    0
);
dispatch_queue_t queue = dispatch_queue_create("com.example.queue", attr);
```

### 2. 避免死锁
```objc
// 错误示例
dispatch_sync(dispatch_get_main_queue(), ^{
    // 在主队列中同步派发到主队列会导致死锁
});

// 正确示例
dispatch_async(dispatch_get_main_queue(), ^{
    // 异步派发不会导致死锁
});
```

## 注意事项

1. **队列选择**
   - 耗时操作使用后台队列
   - UI 操作必须在主队列
   - 合理使用串行和并行队列

2. **内存管理**
   - Block 中的对象引用
   - 避免循环引用
   - 注意 Block 的内存泄漏

3. **线程安全**
   - 使用合适的同步机制
   - 避免过度同步
   - 注意死锁问题

## 面试要点

1. GCD 的基本概念和优势是什么？
- GCD 是 Apple 开发的多核编程解决方案
- 优势:
  - 自动管理线程池
  - 简化并发编程
  - 提供系统级优化
  - 代码简洁易用
  - 自动平衡系统负载

2. 串行队列和并行队列的区别？
- 串行队列:
  - 任务按顺序执行
  - 同一时间只执行一个任务
  - 适合有序任务处理
- 并行队列:
  - 多个任务可同时执行
  - 执行顺序不确定
  - 适合独立任务处理

3. dispatch_sync 和 dispatch_async 的区别？
- dispatch_sync:
  - 同步执行,会阻塞当前线程
  - 等待任务完成才返回
  - 容易造成死锁
- dispatch_async:
  - 异步执行,不会阻塞
  - 立即返回继续执行
  - 适合耗时操作

4. 如何使用 GCD 实现线程同步？
- dispatch_semaphore
- dispatch_barrier_async
- dispatch_group
- dispatch_sync
- 串行队列

5. GCD 中的死锁如何产生？如何避免？
- 产生原因:
  - 在当前队列中同步派发到自身
  - 多个线程互相等待资源
  - 嵌套使用 dispatch_sync
- 避免方法:
  - 使用异步派发
  - 避免在当前队列同步派发
  - 合理设计同步逻辑

6. dispatch_group 的使用场景？
- 多个异步任务协调
- 批量下载任务
- 并发任务合并
- 复杂任务流程控制
- 等待多个任务完成

7. GCD 的性能优化方案有哪些？
- 合理选择队列类型
- 设置合适的 QoS
- 避免频繁创建队列
- 重用dispatch对象
- 避免过度并发
- 合理使用同步机制

## 相关资源

- [Grand Central Dispatch (GCD) Reference](https://developer.apple.com/documentation/dispatch)
- [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html)
- [WWDC: Concurrent Programming With GCD](https://developer.apple.com/videos/play/wwdc2012/712/) 