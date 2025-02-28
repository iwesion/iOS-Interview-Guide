# Block 原理

## 基本概念

Block 是 C 语言的扩充功能，它可以实现闭包。主要包括：
- Block 本质：带有自动变量的匿名函数
- 变量捕获：值捕获和指针捕获
- 内存管理：栈区、堆区和全局区
- 循环引用：Block 的内存泄漏问题

## Block 结构

### 1. Block 类型
```objc
// Block 的三种类型
int main() {
    // 全局 Block
    void (^globalBlock)(void) = ^{
        printf("Global Block\n");
    };
    
    // 栈 Block
    int val = 10;
    void (^stackBlock)(void) = ^{
        printf("Stack Block: %d\n", val);
    };
    
    // 堆 Block
    void (^heapBlock)(void) = [^{
        printf("Heap Block: %d\n", val);
    } copy];
}
```

### 2. 变量捕获
```objc
// 变量捕获机制
@implementation BlockCapture

- (void)captureVariables {
    // 局部变量
    int localVal = 10;
    void (^block)(void) = ^{
        NSLog(@"Local Value: %d", localVal);
    };
    
    // 静态变量
    static int staticVal = 20;
    void (^staticBlock)(void) = ^{
        NSLog(@"Static Value: %d", staticVal);
        staticVal = 30;  // 可修改
    };
    
    // 实例变量
    void (^instanceBlock)(void) = ^{
        NSLog(@"Instance Value: %d", _instanceVal);
    };
}

@end
```

## 内存管理

### 1. Block 的复制
```objc
// Block 的 copy 操作
@implementation BlockCopy

- (void)copyBlock {
    // MRC 下需要手动 copy
    int val = 10;
    void (^block)(void) = [[^{
        NSLog(@"Value: %d", val);
    } copy] autorelease];
    
    // ARC 下自动 copy
    NSArray *array = @[@1, @2, @3];
    void (^arcBlock)(void) = ^{
        NSLog(@"Array: %@", array);
    };
    // 赋值给 __strong 修饰的变量时，自动 copy
}

@end
```

### 2. 循环引用
```objc
// 循环引用问题
@implementation BlockRetainCycle

- (void)causeRetainCycle {
    // 产生循环引用
    self.block = ^{
        NSLog(@"%@", self);
    };
    
    // 解决方案1：__weak
    __weak typeof(self) weakSelf = self;
    self.block = ^{
        NSLog(@"%@", weakSelf);
    };
    
    // 解决方案2：__block (MRC)
    __block typeof(self) blockSelf = self;
    self.block = ^{
        NSLog(@"%@", blockSelf);
        blockSelf = nil;
    };
}

@end
```

## Block 实现

### 1. Block 的底层结构
```objc
// Block 的 C++ 实现
struct Block_literal_1 {
    void *isa;  // Block 的类型
    int flags;  // Block 的标志
    int reserved;
    void (*invoke)(void *, ...);  // 函数指针
    struct Block_descriptor_1 *descriptor;
    // 捕获的变量
};

// Block 描述
struct Block_descriptor_1 {
    unsigned long int reserved;
    unsigned long int size;
    void (*copy)(void *dst, void *src);
    void (*dispose)(void *);
};
```

### 2. 变量捕获实现
```objc
// 变量捕获的实现
int main() {
    int val = 10;
    const char *text = "Hello";
    
    void (^block)(void) = ^{
        printf("%d %s\n", val, text);
    };
    
    // 转换为 C++ 后的结构
    struct Block_literal_1 {
        void *isa;
        int flags;
        int reserved;
        void (*invoke)(struct Block_literal_1 *);
        struct Block_descriptor_1 *descriptor;
        int val;  // 捕获的变量
        const char *text;  // 捕获的变量
    };
}
```

## 实践应用

### 1. 常见用法
```objc
// Block 的使用场景
@implementation BlockUsage

// 属性声明
@property (nonatomic, copy) void (^completionBlock)(void);

// 方法参数
- (void)performWithBlock:(void(^)(void))block {
    if (block) {
        block();
    }
}

// 异步回调
- (void)asyncOperation:(void(^)(NSString *result))completion {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 异步操作
        NSString *result = @"Done";
        
        dispatch_async(dispatch_get_main_queue(), ^{
            completion(result);
        });
    });
}

@end
```

### 2. 注意事项
```objc
// Block 使用注意点
@implementation BlockNotice

// 避免循环引用
- (void)avoidRetainCycle {
    @weakify(self);
    self.block = ^{
        @strongify(self);
        [self doSomething];
    };
}

// 及时释放
- (void)releaseBlock {
    if (self.block) {
        void (^tempBlock)(void) = self.block;
        self.block = nil;
        tempBlock();
    }
}

@end
```

## 注意事项

1. **内存管理**
   - 注意循环引用
   - 合理使用 __weak
   - Block 的 copy

2. **变量捕获**
   - 值捕获和指针捕获
   - 修改外部变量
   - 对象类型的捕获

3. **使用技巧**
   - 避免过度使用
   - 注意性能影响
   - 合理设计接口

## 面试要点

1. Block 的本质是什么？
- Block 是带有自动变量的匿名函数
- 本质是 OC 对象，具有 isa 指针
- 包含函数指针和变量捕获列表
- 可以在不同的上下文中执行
- 支持闭包特性

2. Block 的变量捕获机制？
- 局部变量：值捕获，无法修改
- 静态变量：指针捕获，可以修改
- 全局变量：直接访问，不需要捕获
- 对象类型：连同所有权修饰符一起捕获
- __block 变量：创建包装对象，可修改

3. Block 的内存管理？
- 全局 Block：.data 区，不需要管理
- 栈 Block：函数返回时销毁
- 堆 Block：需要手动管理引用计数
- copy 操作会将栈 Block 拷贝到堆
- ARC 下会自动 copy 到堆区

4. 如何解决循环引用？
- 使用 __weak 修饰 self
- @weakify/@strongify 宏
- __block 配合 nil 置空
- 使用局部变量打破循环
- 合理设计对象生命周期

5. Block 的底层实现？
- 编译器会将 Block 转换为结构体
- 包含 isa、函数指针、描述信息
- 变量捕获列表存储在结构体中
- 不同类型 Block 有不同的 isa
- Runtime 提供相关支持函数

6. Block 的最佳实践？
- 避免在 Block 中直接使用 self
- 注意变量的内存管理
- 合理使用类型定义
- 避免过度嵌套
- 注意 Block 的释放时机

7. Block 的性能优化？
- 避免频繁创建临时 Block
- 合理使用 copy 和 release
- 减少不必要的变量捕获
- 避免大量数据的值捕获
- 注意 Block 的内存占用

## 相关资源

- [Working with Blocks](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html)
- [A Short Practical Guide to Blocks](https://developer.apple.com/library/archive/featuredarticles/Short_Practical_Guide_Blocks/index.html)
- [Block Implementation Specification](https://clang.llvm.org/docs/Block-ABI-Apple.html) 