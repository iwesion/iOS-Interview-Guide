# Method Swizzling 原理

## 基本概念

Method Swizzling 是利用 Objective-C Runtime 特性来交换两个方法的实现的技术。这是一种在运行时改变既有方法行为的强大技术。

## 实现原理

### 核心API
```objc
// 核心交换方法API
void method_exchangeImplementations(Method m1, Method m2);

// 获取方法
Method class_getInstanceMethod(Class aClass, SEL aSelector);
Method class_getClassMethod(Class aClass, SEL aSelector);
```

### 标准实现示例

```objc
@implementation UIViewController (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);
        
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
        // 先尝试添加方法实现
        BOOL didAddMethod = class_addMethod(class,
                                          originalSelector,
                                          method_getImplementation(swizzledMethod),
                                          method_getTypeEncoding(swizzledMethod));
        
        if (didAddMethod) {
            class_replaceMethod(class,
                              swizzledSelector,
                              method_getImplementation(originalMethod),
                              method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated]; // 调用原实现
    // 添加新的实现...
    [Analytics logPageView:NSStringFromClass([self class])];
}

@end
```

## 使用场景

1. 无侵入性统计埋点
2. AOP（面向切面编程）
3. 页面访问统计
4. Method Hook
5. Crash 防护

## 最佳实践

### 1. 命名规范
```objc
// 使用前缀避免命名冲突
- (void)xxx_originalMethod {
    // 实现
}
```

### 2. 安全性考虑
```objc
+ (void)load {
    // 使用dispatch_once确保只执行一次
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // Method Swizzling 代码
    });
}
```

### 3. 方法交换检查
```objc
if (!originalMethod || !swizzledMethod) {
    NSLog(@"Method Swizzling Failed!");
    return;
}
```

## 实践案例

### 1. 防止数组越界崩溃
```objc
@implementation NSArray (Safe)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = NSClassFromString(@"__NSArrayI");
        Method originalMethod = class_getInstanceMethod(class, @selector(objectAtIndex:));
        Method swizzledMethod = class_getInstanceMethod(class, @selector(safe_objectAtIndex:));
        method_exchangeImplementations(originalMethod, swizzledMethod);
    });
}

- (id)safe_objectAtIndex:(NSUInteger)index {
    if (index >= self.count) {
        return nil;
    }
    return [self safe_objectAtIndex:index];
}

@end
```

### 2. 统一页面埋点
```objc
@implementation UIViewController (Analytics)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [self swizzleViewDidAppear];
    });
}

- (void)analytics_viewDidAppear:(BOOL)animated {
    [self analytics_viewDidAppear:animated];
    [Analytics logPageView:NSStringFromClass([self class])];
}

@end
```

## 注意事项

1. **执行时机**
   - 必须在+load方法中执行
   - 不要在+initialize中执行

2. **线程安全**
   - Runtime操作需要考虑线程安全
   - 使用dispatch_once确保只执行一次

3. **继承关系**
   - 注意父类方法的影响
   - 考虑是否影响子类方法

4. **调试难度**
   - Method Swizzling会增加调试难度
   - 建议添加日志跟踪

## 面试要点

1. Method Swizzling的原理是什么？
   - 利用Runtime API交换两个方法的实现(IMP)
   - 主要使用method_exchangeImplementations函数
   - 实质是修改类的方法列表中method对应的IMP指针

2. 为什么要在+load方法中执行？
   - +load在类加载时自动调用,比较早
   - +initialize可能被调用多次,不安全
   - 确保在方法被使用前完成交换
   - 避免出现并发问题

3. Method Swizzling的使用场景有哪些？
   - AOP切面编程(统一埋点、日志等)
   - 为已有方法添加功能(方法增强)
   - Hook系统方法
   - 防护系统API(数组越界等)

4. 如何确保Method Swizzling的安全性？
   - 使用dispatch_once确保只交换一次
   - 检查方法是否存在
   - 注意命名冲突
   - 保持方法签名一致
   - 考虑线程安全问题

5. Method Swizzling有什么注意事项？
   - 注意命名规范,避免冲突
   - 需要考虑继承链的影响
   - 不建议在生产环境大量使用
   - 增加了程序的不确定性
   - 可能影响调试和维护

## 相关资源

- [Apple Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)
- [NSHipster: Method Swizzling](https://nshipster.com/method-swizzling/)
- [Objective-C Runtime 运行时](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html)
