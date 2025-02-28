# 线程安全

## 基本概念

线程安全是指在多线程环境下，代码能够正确地处理多个线程同时访问共享资源的情况，不会出现数据竞争、死锁等问题。

## 常见问题

### 1. 数据竞争
```objc
// 不安全的示例
@interface Counter : NSObject
@property (nonatomic, assign) NSInteger count;
@end

@implementation Counter
- (void)increment {
    self.count++; // 非原子操作，可能导致数据竞争
}
@end
```

### 2. 死锁
```objc
// 死锁示例
dispatch_queue_t queueA = dispatch_queue_create("com.example.queueA", NULL);
dispatch_queue_t queueB = dispatch_queue_create("com.example.queueB", NULL);

dispatch_async(queueA, ^{
    dispatch_sync(queueB, ^{
        // 任务1
    });
});

dispatch_async(queueB, ^{
    dispatch_sync(queueA, ^{
        // 任务2
    });
});
```

## 解决方案

### 1. 同步锁
```objc
@interface ThreadSafeArray : NSObject
@property (nonatomic, strong) NSMutableArray *array;
@property (nonatomic, strong) NSLock *lock;
@end

@implementation ThreadSafeArray

- (void)addObject:(id)object {
    [self.lock lock];
    [self.array addObject:object];
    [self.lock unlock];
}

- (id)objectAtIndex:(NSUInteger)index {
    [self.lock lock];
    id object = [self.array objectAtIndex:index];
    [self.lock unlock];
    return object;
}
@end
```

### 2. 串行队列
```objc
@interface ThreadSafeObject : NSObject
@property (nonatomic, strong) dispatch_queue_t queue;
@property (nonatomic, strong) NSMutableDictionary *dictionary;
@end

@implementation ThreadSafeObject

- (void)setObject:(id)object forKey:(NSString *)key {
    dispatch_sync(self.queue, ^{
        [self.dictionary setObject:object forKey:key];
    });
}

- (id)objectForKey:(NSString *)key {
    __block id result = nil;
    dispatch_sync(self.queue, ^{
        result = [self.dictionary objectForKey:key];
    });
    return result;
}
@end
```

### 3. 原子操作
```objc
// 使用 atomic 属性
@property (atomic, strong) NSString *name;

// 使用 OSAtomic 函数
int32_t __attribute__((aligned(4))) value = 0;
OSAtomicIncrement32(&value);
```

## 实践案例

### 1. 线程安全的单例
```objc
@implementation Singleton

+ (instancetype)sharedInstance {
    static Singleton *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[self alloc] init];
    });
    return instance;
}

@end
```

### 2. 线程安全的缓存
```objc
@interface ThreadSafeCache : NSObject

@property (nonatomic, strong) dispatch_queue_t queue;
@property (nonatomic, strong) NSCache *cache;

@end

@implementation ThreadSafeCache

- (void)setObject:(id)object forKey:(NSString *)key {
    dispatch_barrier_async(self.queue, ^{
        [self.cache setObject:object forKey:key];
    });
}

- (id)objectForKey:(NSString *)key {
    __block id result = nil;
    dispatch_sync(self.queue, ^{
        result = [self.cache objectForKey:key];
    });
    return result;
}

@end
```

## 性能优化

### 1. 读写锁
```objc
// 使用 pthread_rwlock
pthread_rwlock_t lock;
pthread_rwlock_init(&lock, NULL);

// 读操作
pthread_rwlock_rdlock(&lock);
// 读取数据
pthread_rwlock_unlock(&lock);

// 写操作
pthread_rwlock_wrlock(&lock);
// 写入数据
pthread_rwlock_unlock(&lock);
```

### 2. 条件锁
```objc
@interface DataBuffer : NSObject
@property (nonatomic, strong) NSCondition *condition;
@property (nonatomic, strong) NSMutableArray *buffer;
@end

@implementation DataBuffer

- (void)produce:(id)data {
    [self.condition lock];
    [self.buffer addObject:data];
    [self.condition signal];
    [self.condition unlock];
}

- (id)consume {
    [self.condition lock];
    while (self.buffer.count == 0) {
        [self.condition wait];
    }
    id data = self.buffer.firstObject;
    [self.buffer removeObjectAtIndex:0];
    [self.condition unlock];
    return data;
}

@end
```

## 注意事项

1. **选择合适的同步机制**
   - 根据场景选择合适的锁
   - 避免过度同步
   - 注意性能影响

2. **避免死锁**
   - 保持一致的加锁顺序
   - 避免循环依赖
   - 使用超时机制

3. **性能考虑**
   - 最小化临界区
   - 使用细粒度锁
   - 考虑无锁算法

## 面试要点

1. 什么是线程安全？
- 多线程环境下能正确处理共享资源访问
- 避免数据竞争和不一致
- 保证操作的原子性、可见性和有序性
- 常见实现:同步锁、原子操作、线程隔离

2. 常见的线程同步方式有哪些？
- @synchronized 
- NSLock
- pthread_mutex
- dispatch_semaphore
- OSSpinLock(已弃用)
- pthread_rwlock
- NSCondition
- NSRecursiveLock

3. 如何避免死锁？
- 避免循环等待
- 保持一致的加锁顺序
- 使用 tryLock 机制
- 设置超时时间
- 减少锁的粒度
- 使用队列而不是锁

4. atomic 属性是否能保证线程安全？
- atomic 只保证属性的 getter/setter 是原子操作
- 不能保证属性相关的复合操作的线程安全
- 不能保证数组、字典等集合类的线程安全
- 通常需要额外的同步机制

5. 如何实现线程安全的集合类？
- 使用同步锁包装所有操作
- 使用 dispatch_queue 进行同步
- 使用 @synchronized 
- 考虑读写锁提高性能
- 注意异常处理和资源释放

6. 读写锁和互斥锁的区别？
- 读写锁允许多个读操作并发
- 写操作时互斥访问
- 适合读多写少的场景
- 实现复杂度较高
- 需要正确处理优先级

7. 如何处理多线程资源竞争？
- 使用同步机制
- 避免共享可变状态
- 使用线程安全的数据结构
- 采用消息队列模式
- 实现生产者消费者模式
- 考虑无锁算法

1. 什么是线程安全？如何实现？
2. 常见的线程同步方式有哪些？
3. 如何避免死锁？
4. atomic 属性是否能保证线程安全？
5. 如何实现一个线程安全的集合类？
6. 读写锁和互斥锁的区别？
7. 如何处理多线程环境下的资源竞争？

## 相关资源

- [Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html)
- [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html)
- [Lock Patterns](https://www.objc.io/issues/2-concurrency/concurrency-apis-and-pitfalls/) 