# NSOperation 详解

## 基本概念

NSOperation 是 Apple 提供的一个面向对象的多线程解决方案。它是对 GCD 的封装，提供了更多的功能，如取消、优先级、依赖关系等。

## 核心组件

### 1. NSOperation
```objc
// 自定义 Operation
@interface CustomOperation : NSOperation

@property (nonatomic, copy) void (^operationBlock)(void);

@end

@implementation CustomOperation

- (void)main {
    if (self.isCancelled) return;
    
    if (self.operationBlock) {
        self.operationBlock();
    }
}

@end
```

### 2. NSOperationQueue
```objc
// 创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.maxConcurrentOperationCount = 3;

// 主队列
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
```

## 常用功能

### 1. 操作依赖
```objc
NSOperation *operation1 = [NSOperation new];
NSOperation *operation2 = [NSOperation new];

// 添加依赖
[operation2 addDependency:operation1];

// 移除依赖
[operation2 removeDependency:operation1];
```

### 2. 优先级设置
```objc
operation.queuePriority = NSOperationQueuePriorityHigh;
operation.qualityOfService = NSQualityOfServiceUserInitiated;
```

### 3. 取消操作
```objc
// 取消单个操作
[operation cancel];

// 取消队列中所有操作
[queue cancelAllOperations];
```

## 实践案例

### 1. 图片下载
```objc
@interface ImageDownloadOperation : NSOperation

@property (nonatomic, strong) NSURL *imageURL;
@property (nonatomic, copy) void (^completionBlock)(UIImage *image);

@end

@implementation ImageDownloadOperation

- (void)main {
    if (self.isCancelled) return;
    
    NSData *imageData = [NSData dataWithContentsOfURL:self.imageURL];
    if (self.isCancelled) return;
    
    UIImage *image = [UIImage imageWithData:imageData];
    if (self.isCancelled) return;
    
    if (self.completionBlock) {
        self.completionBlock(image);
    }
}

@end
```

### 2. 并发操作
```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.maxConcurrentOperationCount = 3;

for (NSURL *url in imageURLs) {
    ImageDownloadOperation *operation = [[ImageDownloadOperation alloc] init];
    operation.imageURL = url;
    operation.completionBlock = ^{
        // 处理下载完成的图片
    };
    [queue addOperation:operation];
}
```

### 3. 操作依赖链
```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];

NSOperation *downloadOperation = [NSOperation new];
NSOperation *processOperation = [NSOperation new];
NSOperation *saveOperation = [NSOperation new];

[processOperation addDependency:downloadOperation];
[saveOperation addDependency:processOperation];

[queue addOperations:@[downloadOperation, processOperation, saveOperation] 
    waitUntilFinished:NO];
```

## 性能优化

### 1. 队列优化
```objc
// 设置合适的并发数
queue.maxConcurrentOperationCount = NSProcessorCount;

// 设置合理的优先级
operation.queuePriority = NSOperationQueuePriorityNormal;
```

### 2. 内存管理
```objc
// 在合适的时机取消操作
- (void)dealloc {
    [self.operationQueue cancelAllOperations];
}
```

## 注意事项

1. **状态管理**
   - 正确处理取消状态
   - 注意操作的完成状态
   - 合理设置依赖关系

2. **内存管理**
   - 注意循环引用
   - 及时清理不需要的操作
   - 控制并发数量

3. **线程安全**
   - 注意数据竞争
   - 使用同步机制
   - 避免死锁

## 面试要点

1. NSOperation 和 GCD 的区别是什么？
2. NSOperation 的状态有哪些？
3. 如何实现操作依赖？
4. NSOperation 的优先级如何设置？
5. 如何取消正在执行的操作？
6. NSOperation 和 NSOperationQueue 的关系？
7. 如何实现线程安全的 NSOperation？

## 相关资源

- [NSOperation Class Reference](https://developer.apple.com/documentation/foundation/nsoperation)
- [NSOperationQueue Class Reference](https://developer.apple.com/documentation/foundation/nsoperationqueue)
- [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html) 