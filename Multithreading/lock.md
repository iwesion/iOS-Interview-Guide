# 锁机制

## 基本概念

锁是用于多线程同步的一种机制，用于保护共享资源，确保在同一时间只有一个线程可以访问这些资源。iOS 提供了多种锁机制，每种都有其特定的使用场景。

## 锁的类型

### 1. @synchronized
```objc
// 最基本的锁
@synchronized(self) {
    // 临界区代码
}
```

### 2. NSLock
```objc
NSLock *lock = [[NSLock alloc] init];
[lock lock];
// 临界区代码
[lock unlock];
```

### 3. NSRecursiveLock
```objc
NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
[lock lock];
// 可重入的临界区代码
[lock unlock];
```

### 4. pthread_mutex
```objc
pthread_mutex_t mutex;
pthread_mutex_init(&mutex, NULL);
pthread_mutex_lock(&mutex);
// 临界区代码
pthread_mutex_unlock(&mutex);
```

### 5. OSSpinLock (已弃用)
```objc
// 自旋锁，不再推荐使用
OSSpinLock lock = OS_SPINLOCK_INIT;
OSSpinLockLock(&lock);
// 临界区代码
OSSpinLockUnlock(&lock);
```

### 6. os_unfair_lock
```objc
// OSSpinLock 的替代品
os_unfair_lock lock = OS_UNFAIR_LOCK_INIT;
os_unfair_lock_lock(&lock);
// 临界区代码
os_unfair_lock_unlock(&lock);
```

## 性能对比

### 1. 性能测试
```objc
- (void)testLockPerformance {
    // @synchronized
    CFAbsoluteTime time1 = CFAbsoluteTimeGetCurrent();
    for (int i = 0; i < 1000000; i++) {
        @synchronized(self) {
            // 空操作
        }
    }
    NSLog(@"@synchronized: %f", CFAbsoluteTimeGetCurrent() - time1);
    
    // NSLock
    NSLock *lock = [[NSLock alloc] init];
    CFAbsoluteTime time2 = CFAbsoluteTimeGetCurrent();
    for (int i = 0; i < 1000000; i++) {
        [lock lock];
        // 空操作
        [lock unlock];
    }
    NSLog(@"NSLock: %f", CFAbsoluteTimeGetCurrent() - time2);
}
```

### 2. 选择建议
- 简单场景：@synchronized
- 一般场景：NSLock
- 递归调用：NSRecursiveLock
- 高性能要求：os_unfair_lock
- 复杂场景：pthread_mutex

## 实践案例

### 1. 线程安全的数组
```objc
@interface ThreadSafeArray : NSObject
@property (nonatomic, strong) NSMutableArray *array;
@property (nonatomic, strong) NSLock *lock;
@end

@implementation ThreadSafeArray

- (instancetype)init {
    if (self = [super init]) {
        _array = [NSMutableArray array];
        _lock = [[NSLock alloc] init];
    }
    return self;
}

- (void)addObject:(id)object {
    [self.lock lock];
    [self.array addObject:object];
    [self.lock unlock];
}

- (void)removeObject:(id)object {
    [self.lock lock];
    [self.array removeObject:object];
    [self.lock unlock];
}

@end
```

### 2. 递归锁使用
```objc
@interface RecursiveOperation : NSObject
@property (nonatomic, strong) NSRecursiveLock *lock;
@end

@implementation RecursiveOperation

- (void)recursiveMethod:(NSInteger)count {
    [self.lock lock];
    if (count > 0) {
        [self recursiveMethod:count - 1];
    }
    [self.lock unlock];
}

@end
```

### 3. 条件锁
```objc
@interface ConditionLock : NSObject
@property (nonatomic, strong) NSConditionLock *lock;
@end

@implementation ConditionLock

- (void)producer {
    [self.lock lock];
    // 生产数据
    [self.lock unlockWithCondition:1];
}

- (void)consumer {
    [self.lock lockWhenCondition:1];
    // 消费数据
    [self.lock unlock];
}

@end
```

## 注意事项

1. **死锁预防**
   - 避免循环等待
   - 保持一致的加锁顺序
   - 使用超时机制

2. **性能考虑**
   - 选择合适的锁类型
   - 最小化临界区
   - 避免锁的滥用

3. **使用规范**
   - 成对使用 lock/unlock
   - 注意异常处理
   - 避免嵌套锁

## 面试要点

1. iOS 中有哪些锁？各自的特点是什么？
- @synchronized: 最简单的锁,性能一般,使用方便
- NSLock: 基本互斥锁,性能好,需手动管理
- NSRecursiveLock: 递归锁,允许同一线程多次加锁
- pthread_mutex: 底层互斥锁,性能最好,使用复杂
- os_unfair_lock: 自旋锁替代品,性能好,适合短任务

2. @synchronized 的实现原理？
- 基于互斥锁实现
- 以传入对象的内存地址作为 key
- 维护一个全局的哈希表来管理锁
- 自动管理加锁和解锁
- 异常时自动释放锁

3. 自旋锁和互斥锁的区别？
- 自旋锁:
  - 忙等待,不释放 CPU
  - 适合短时间等待
  - 开销小但耗 CPU
- 互斥锁:
  - 线程休眠,释放 CPU
  - 适合长时间等待
  - 开销大但节省资源

4. 如何避免死锁？
- 按固定顺序申请锁
- 使用 tryLock 机制
- 设置加锁超时
- 避免嵌套加锁
- 及时释放锁

5. 不同锁的性能对比？
从高到低:
- os_unfair_lock
- pthread_mutex
- NSLock
- @synchronized
- NSRecursiveLock
- NSConditionLock

6. 什么场景下使用递归锁？
- 方法递归调用
- 嵌套加锁场景
- 同一线程多次加锁
- 避免自己锁死自己
- 复杂的层级调用

7. 条件锁的使用场景？
- 生产者消费者模式
- 线程通信
- 任务依赖管理
- 状态同步
- 资源控制

## 相关资源

- [Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html)
- [Synchronization](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/ThreadSafety/ThreadSafety.html)
- [Lock Patterns in iOS](https://www.objc.io/issues/2-concurrency/concurrency-apis-and-pitfalls/) 