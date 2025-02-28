# Runtime消息发送机制的完整流程

## 基本概念

Objective-C 的动态特性主要体现在其消息发送机制上。当我们调用一个对象的方法时，实际上是向该对象发送了一条消息。这个过程由 Runtime 系统负责处理，而不是像 C++ 那样在编译时就确定函数调用。

## 消息发送的基本流程图解

````mermaid
sequenceDiagram
    participant 应用程序
    participant objc_msgSend
    participant 类缓存
    participant 类方法列表
    participant 父类方法
    participant 动态方法解析
    participant 消息转发
    
    应用程序->>objc_msgSend: [obj method:param]
    
    objc_msgSend->>类缓存: 查找缓存
    
    alt 缓存命中
        类缓存-->>objc_msgSend: 返回IMP
    else 缓存未命中
        objc_msgSend->>类方法列表: 查找方法
        
        alt 方法找到
            类方法列表-->>objc_msgSend: 返回IMP并缓存
        else 方法未找到
            objc_msgSend->>父类方法: 沿继承链查找
            
            alt 父类方法找到
                父类方法-->>objc_msgSend: 返回IMP并缓存
            else 父类方法未找到
                objc_msgSend->>动态方法解析: 调用resolveInstanceMethod:
                
                alt 动态添加方法成功
                    动态方法解析-->>objc_msgSend: 重新发送消息
                else 动态添加方法失败
                    objc_msgSend->>消息转发: 进入消息转发流程
                end
            end
        end
    end
    
    objc_msgSend-->>应用程序: 执行找到的方法实现
````

## 消息转发的详细流程

````mermaid
flowchart TD
    A[消息无法正常处理] --> B[调用forwardingTargetForSelector:]
    B --> C{是否返回非nil对象?}
    C -->|是| D[向新对象发送消息]
    C -->|否| E[调用methodSignatureForSelector:]
    E --> F{是否返回方法签名?}
    F -->|是| G[创建NSInvocation并调用forwardInvocation:]
    F -->|否| H[调用doesNotRecognizeSelector:]
    G --> I[在forwardInvocation:中处理消息]
    H --> J[抛出异常]
````

## 消息发送的完整流程

````mermaid
graph TD
    A[消息发送: objc_msgSend] --> B{在接收者类的缓存中查找?}
    B -->|找到| C[调用方法实现IMP]
    B -->|未找到| D{在接收者类的方法列表中查找?}
    D -->|找到| E[将方法加入缓存]
    E --> C
    D -->|未找到| F{在父类中递归查找?}
    F -->|找到| E
    F -->|未找到| G[动态方法解析]
    G --> H{resolveInstanceMethod:/resolveClassMethod:?}
    H -->|解析成功| I[重新进入消息发送流程]
    H -->|解析失败| J[消息转发]
    J --> K{forwardingTargetForSelector:?}
    K -->|返回新对象| L[向新对象发送消息]
    K -->|返回nil| M{methodSignatureForSelector:?}
    M -->|返回签名| N[forwardInvocation:]
    M -->|返回nil| O[doesNotRecognizeSelector:]
    O --> P[抛出异常: 方法未找到]
````

## 方法查找的详细过程

````mermaid
flowchart LR
    A[objc_msgSend] --> B[在类缓存中查找]
    B --> C{找到方法?}
    C -->|是| D[返回IMP]
    C -->|否| E[在类方法列表中查找]
    E --> F{找到方法?}
    F -->|是| G[缓存方法并返回IMP]
    F -->|否| H[在父类中查找]
    H --> I{找到方法?}
    I -->|是| G
    I -->|否| J{是否已到根类?}
    J -->|否| H
    J -->|是| K[进入动态解析阶段]
````

## 核心数据结构

````mermaid
classDiagram
    class objc_object {
        +isa_t isa
    }
    
    class objc_class {
        +Class superclass
        +cache_t cache
        +class_data_bits_t bits
        +method_list_t methods
        +protocol_list_t protocols
        +property_list_t properties
    }
    
    class method_t {
        +SEL name
        +const char* types
        +IMP imp
    }
    
    class cache_t {
        -bucket_t* _buckets
        -mask_t _mask
        -mask_t _occupied
    }
    
    objc_object <|-- objc_class
    objc_class --> cache_t
    objc_class --> method_t
````

## 消息发送的代码示例

````objc
// 原始的Objective-C代码
[object method:parameter];

// 被编译器转换为
objc_msgSend(object, @selector(method:), parameter);
````

## 动态方法解析示例

````objc
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(dynamicMethod)) {
        class_addMethod(self, sel, (IMP)dynamicMethodIMP, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}

void dynamicMethodIMP(id self, SEL _cmd) {
    NSLog(@"动态添加的方法实现");
}
````

## 消息转发示例

````objc
// 第二步：快速转发
- (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(unknownMethod)) {
        return alternativeObject; // 将消息转发给其他对象
    }
    return [super forwardingTargetForSelector:aSelector];
}

// 第三步：常规转发
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    if (aSelector == @selector(unknownMethod)) {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    return [super methodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if (anInvocation.selector == @selector(unknownMethod)) {
        // 处理未知消息
        NSLog(@"处理未知消息");
    } else {
        [super forwardInvocation:anInvocation];
    }
}
````

## 性能考虑

1. 方法缓存机制大大提高了消息发送的效率
2. 消息转发机制虽然强大，但会带来性能开销
3. 频繁调用的方法可以考虑使用IMP缓存或C函数调用优化性能

## 总结

Objective-C的消息发送机制是其动态性的核心，通过Runtime系统实现。完整流程包括：方法查找、动态方法解析和消息转发三个阶段，为开发者提供了极大的灵活性，使得诸如Method Swizzling、动态添加方法等高级技术成为可能。
