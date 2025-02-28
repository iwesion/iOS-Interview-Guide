# Weak 实现原理

## 基本概念

weak 是 Objective-C 中的一种所有权修饰符，用于解决循环引用问题。weak 引用不会增加对象的引用计数，当对象被释放时，weak 指针会自动置为 nil。

## 实现原理

### 1. 数据结构
```objc
// weak 表结构
struct weak_table_t {
    weak_entry_t *weak_entries;
    size_t num_entries;
    uintptr_t mask;
    uintptr_t max_hash_displacement;
};

// weak 引用记录
struct weak_entry_t {
    DisguisedPtr<objc_object> referent;
    union {
        struct {
            weak_referrer_t *referrers;
            uintptr_t out_of_line : 1;
            uintptr_t num_refs : PTR_MINUS_1;
            uintptr_t mask;
            uintptr_t max_hash_displacement;
        };
        struct {
            weak_referrer_t inline_referrers[WEAK_INLINE_COUNT];
        };
    };
};
```

### 2. 核心操作
```objc
// 设置 weak 引用
id objc_initWeak(id *location, id newObj);

// 销毁 weak 引用
void objc_destroyWeak(id *location);

// 加载 weak 引用
id objc_loadWeakRetained(id *location);
```

## 工作流程

### 1. 初始化过程
```objc
{
    __weak id weakObj = obj;
    // 1. 检查对象是否支持弱引用
    // 2. 创建弱引用表项
    // 3. 将 weak 指针加入弱引用表
}
```

### 2. 对象销毁时
```objc
- (void)dealloc {
    // 1. 从弱引用表中获取所有 weak 指针
    // 2. 将所有 weak 指针置为 nil
    // 3. 从弱引用表中移除记录
}
```

## 实践案例

### 1. 解决循环引用
```objc
@interface ViewController ()
@property (nonatomic, copy) void (^block)(void);
@end

@implementation ViewController
- (void)setup {
    __weak typeof(self) weakSelf = self;
    self.block = ^{
        [weakSelf doSomething];
    };
}
@end
```

### 2. 代理模式
```objc
@interface MyView : UIView
@property (nonatomic, weak) id<MyViewDelegate> delegate;
@end

@implementation MyView
- (void)someAction {
    if ([self.delegate respondsToSelector:@selector(viewDidSomething:)]) {
        [self.delegate viewDidSomething:self];
    }
}
@end
```

## 性能优化

### 1. SideTable 优化
```objc
// 使用分离锁减少锁竞争
static StripedMap<SideTable>& SideTables() {
    return *reinterpret_cast<StripedMap<SideTable>*>(SideTableBuf);
}
```

### 2. 内联优化
```objc
// 小数组优化
#define WEAK_INLINE_COUNT 4
```

## 注意事项

1. **线程安全**
   - weak 操作是线程安全的
   - 使用 SideTable 锁保护

2. **性能开销**
   - weak 比 strong 慢
   - 设置和访问都有额外开销

3. **使用限制**
   - 不是所有对象都支持 weak 引用
   - NSString、NSNumber 等对象不支持

## 面试要点

1. weak 的实现原理是什么？
- 通过 SideTable 哈希表存储对象的 weak 引用
- 对象销毁时通过 weak_clear_no_lock 函数将所有 weak 指针置为 nil
- 使用引用计数和弱引用表实现自动置空
- Runtime 维护全局的 weak 引用表
- 分离锁机制保证线程安全

2. weak 指针的数据结构是怎样的？
- SideTable 结构包含自旋锁、引用计数表和弱引用表
- 弱引用表是哈希表结构
- key 是对象地址，value 是 weak 指针数组
- 使用 DenseMap 优化内存占用
- 支持快速查找和更新操作

3. 对象销毁时 weak 指针是如何处理的？
- dealloc 时调用 weak_clear_no_lock
- 获取对象的 SideTable
- 遍历 weak 指针数组
- 将所有 weak 指针置为 nil
- 从弱引用表中移除对象

4. weak 的性能如何？有什么优化措施？
- 比 strong 慢，需要额外的表操作
- 使用分离锁减少锁竞争
- 小数组优化减少内存分配
- DenseMap 优化内存占用
- 哈希表优化查找性能

5. 什么情况下应该使用 weak？
- 代理模式避免循环引用
- Block 中打破循环引用
- IBOutlet 避免强引用
- 临时引用其他对象
- 父子视图关系中的向上引用

6. weak 和 assign 的区别是什么？
- weak 修饰对象，会自动置空
- assign 修饰基本数据类型，不会置空
- weak 是线程安全的
- assign 可能产生悬垂指针
- weak 有额外的内存开销

7. weak 的线程安全是如何保证的？
- 使用 SideTable 的自旋锁
- 分离锁减少锁竞争
- 读写操作都有锁保护
- 对象销毁时加锁处理
- 使用原子操作保证安全

## 相关资源

- [Objective-C Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html)
- [Advanced Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)
- [Friday Q&A: Zeroing Weak References](https://www.mikeash.com/pyblog/friday-qa-2010-07-16-zeroing-weak-references-in-objective-c.html) 