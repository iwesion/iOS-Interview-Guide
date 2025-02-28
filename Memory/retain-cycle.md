# 循环引用问题

## 基本概念

循环引用(Retain Cycle)是指两个或多个对象互相持有强引用，导致它们都无法被释放的情况。这是内存泄漏的常见原因之一。

## 常见场景

### 1. 对象之间的循环引用
```objc
@interface Person : NSObject
@property (nonatomic, strong) Dog *dog;
@end

@interface Dog : NSObject
@property (nonatomic, strong) Person *owner;
@end

// 产生循环引用
Person *person = [[Person alloc] init];
Dog *dog = [[Dog alloc] init];
person.dog = dog;
dog.owner = person;
```

### 2. Block 中的循环引用
```objc
@interface ViewController ()
@property (nonatomic, copy) void (^block)(void);
@end

@implementation ViewController
- (void)viewDidLoad {
    self.block = ^{
        [self doSomething]; // self 被 block 强引用
    };
}
@end
```

### 3. NSTimer 的循环引用
```objc
@interface ViewController ()
@property (nonatomic, strong) NSTimer *timer;
@end

@implementation ViewController
- (void)viewDidLoad {
    self.timer = [NSTimer scheduledTimerWithTarget:self
                                        selector:@selector(timerFired:)
                                        interval:1.0
                                         repeats:YES];
}
@end
```

## 解决方案

### 1. 使用 weak 引用
```objc
// 对象之间
@interface Dog : NSObject
@property (nonatomic, weak) Person *owner; // 使用 weak
@end

// Block 中
__weak typeof(self) weakSelf = self;
self.block = ^{
    [weakSelf doSomething];
};
```

### 2. weak-strong dance
```objc
__weak typeof(self) weakSelf = self;
self.block = ^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (!strongSelf) return;
    
    [strongSelf doSomething];
    // 这里可以安全地使用 strongSelf
};
```

### 3. NSTimer 的处理
```objc
// 方案1：使用中间对象
@interface TimerTarget : NSObject
@property (nonatomic, weak) id realTarget;
@end

// 方案2：使用 block API (iOS 10+)
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                                            repeats:YES
                                              block:^(NSTimer *timer) {
    [weakSelf timerFired:timer];
}];
```

## 检测方法

### 1. 静态分析
```objc
// 使用 Xcode 的静态分析器
Product -> Analyze
```

### 2. 运行时检测
```objc
// 使用 Instruments 的 Leaks 工具
// 使用 Memory Graph 调试器
```

### 3. 代码审查
```objc
// 检查以下场景：
1. 属性声明中的 strong 关键字
2. block 中捕获 self
3. delegate 属性是否使用 weak
4. NSTimer 的 target
```

## 最佳实践

### 1. 属性声明规范
```objc
// delegate 使用 weak
@property (nonatomic, weak) id<MyDelegate> delegate;

// block 属性注意循环引用
@property (nonatomic, copy) void (^completion)(void);
```

### 2. Block 使用规范
```objc
// 方案1：weak-strong dance
__weak typeof(self) weakSelf = self;
self.block = ^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    if (!strongSelf) return;
    [strongSelf doSomething];
};

// 方案2：直接使用 weak（简单操作）
__weak typeof(self) weakSelf = self;
self.block = ^{
    [weakSelf doSomething];
};
```

### 3. 定时器使用规范
```objc
// iOS 10+ 推荐使用 block 方式
__weak typeof(self) weakSelf = self;
self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                                            repeats:YES
                                              block:^(NSTimer *timer) {
    [weakSelf timerFired:timer];
}];

// 注意在适当时机销毁定时器
- (void)dealloc {
    [self.timer invalidate];
    self.timer = nil;
}
```

## 注意事项

1. **Block 的注意点**
   - 注意 block 属性的声明
   - 注意 block 中对 self 的引用
   - 使用 weak-strong dance 时的线程安全

2. **委托代理**
   - delegate 属性使用 weak
   - 注意代理方法中的循环引用

3. **容器类**
   - 注意数组、字典等容器类中的循环引用
   - 考虑使用 NSHashTable、NSMapTable

## 面试要点

1. 什么是循环引用？如何检测？
- 两个或多个对象互相持有强引用，导致无法释放
- 使用 Xcode Memory Graph 检测
- 使用 Instruments 的 Leaks/Allocations 工具
- Debug Memory Graph 查看对象引用关系
- 使用 deinit 方法验证对象释放

2. 常见的循环引用场景有哪些？
- 对象之间的相互强引用
- Block 捕获 self
- NSTimer 对 target 的强引用
- 代理模式使用 strong
- 通知观察者忘记移除
- 子视图与父视图互相引用

3. weak 和 unowned 的区别？
- weak 引用对象释放后会自动置为 nil
- unowned 引用对象释放后不会置 nil，可能导致崩溃
- weak 更安全但有性能开销
- unowned 适用于确定引用对象存在的场景
- weak 常用于代理、避免循环引用

4. Block 中的循环引用如何解决？
- 使用 weak-strong dance
- 简单场景直接使用 weak
- 使用 __block 修饰符配合手动打破
- 在 block 执行完后将其置为 nil
- 合理设计 block 的生命周期

5. NSTimer 的循环引用为什么特殊？如何解决？
- Timer 会对 target 保持强引用直到 invalidate
- 使用 block 方式创建 Timer（iOS 10+）
- 使用代理对象（proxy）转发消息
- 在合适时机调用 invalidate
- 使用 weak timer 配合 block

6. weak-strong dance 的原理是什么？
- weak 引用避免循环引用
- strong 引用保证执行过程中对象不被释放
- 检查 strongSelf 是否为 nil
- 保证操作的原子性
- 适用于异步和延迟操作

7. 如何在项目中预防循环引用？
- 合理设计对象之间的引用关系
- 使用正确的属性修饰符
- 遵循内存管理最佳实践
- 做好代码审查
- 使用工具定期检查内存泄漏

## 相关资源

- [Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)
- [Advanced Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html)
- [Debugging Memory Issues](https://developer.apple.com/documentation/xcode/diagnosing_memory_thread_and_crash_issues_early) 