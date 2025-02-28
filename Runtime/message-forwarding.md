# 消息发送和转发机制

## 基本原理

Objective-C 的方法调用实际上是向对象发送消息。这个过程主要分为三个阶段：
1. 消息发送
2. 动态方法解析
3. 消息转发

### 1. 消息发送过程

当我们调用一个对象的方法时：
```objc
[object method];
```

实际上会被编译器转换为：
```objc
objc_msgSend(object, @selector(method));
```

消息发送的详细过程：
1. 在对象的 class 中的 cache 中查找方法实现 (缓存查找)
2. 在对象的 class 的方法列表中查找 (当前类查找)
3. 在父类的方法列表中查找，直到 NSObject 类 (继承链查找)
4. 如果还找不到，进入动态方法解析阶段

### 2. 动态方法解析

如果消息发送阶段未找到方法实现，Runtime 会调用以下方法：

```objc
// 对象方法
+ (BOOL)resolveInstanceMethod:(SEL)sel;
// 类方法
+ (BOOL)resolveClassMethod:(SEL)sel;
```

示例实现：
```objc
void dynamicMethodIMP(id self, SEL _cmd) {
    // 方法实现
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(someMethod)) {
        class_addMethod([self class],
                       sel,
                       (IMP)dynamicMethodIMP,
                       "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

### 3. 消息转发

如果动态方法解析返回 NO，则进入消息转发阶段：

#### 3.1 快速转发
```objc
- (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(someMethod)) {
        return alternativeObject; // 返回能响应该方法的对象
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

#### 3.2 完整转发
```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    if (aSelector == @selector(someMethod)) {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    return [super methodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([alternativeObject respondsToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:alternativeObject];
    } else {
        [super forwardInvocation:anInvocation];
    }
}
```

## 实践案例

### 1. 实现多继承效果
```objc
@interface ViewController : UIViewController
@property (nonatomic, strong) Helper *helper;
@end

@implementation ViewController

- (id)forwardingTargetForSelector:(SEL)aSelector {
    if ([self.helper respondsToSelector:aSelector]) {
        return self.helper;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

### 2. 统一异常处理
```objc
@implementation NSObject (ExceptionHandler)

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    @try {
        [super forwardInvocation:anInvocation];
    } @catch (NSException *exception) {
        // 统一处理异常
        [ExceptionHandler handleException:exception];
    }
}

@end
```

### 3. 实现热修复
```objc
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    // 从补丁包中查找方法实现
    IMP imp = [PatchManager implementationForSelector:sel];
    if (imp) {
        class_addMethod(self, sel, imp, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

## 面试要点

1. 消息发送机制的完整流程是什么？
   - 缓存查找 -> 当前类查找 -> 父类查找 -> 动态解析 -> 消息转发

2. 动态方法解析和消息转发的区别？
   - 动态方法解析：为类动态添加方法实现
   - 消息转发：将消息转发给其他对象处理

3. 消息转发的实际应用场景？
   - 实现多继承
   - 统一异常处理
   - 实现热修复
   - AOP（面向切面编程）

4. 如何防止因消息转发引起的性能问题？
   - 缓存转发结果
   - 避免在频繁调用的方法中使用
   - 优先使用快速转发而不是完整转发

## 注意事项

1. 性能考虑
   - 消息转发会带来性能开销
   - 建议缓存转发结果
   - 避免在频繁调用的方法中使用

2. 线程安全
   - 动态添加方法时需要考虑线程安全
   - 转发过程中对象状态的变化

3. 调试难度
   - 消息转发会增加调试难度
   - 建议添加日志跟踪转发过程

## 相关资源

- [Apple Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html)
- [NSObject Class Reference](https://developer.apple.com/documentation/objectivec/nsobject)
- [Understanding the Objective-C Runtime](http://cocoasamurai.blogspot.com/2010/01/understanding-objective-c-runtime.html)
