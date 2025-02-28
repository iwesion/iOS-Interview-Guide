# Class 与 Meta Class

## 基本概念

在 Objective-C 中，所有的类自身也是一个对象，这个类对象是元类（Meta Class）的实例。元类存储类方法的定义，而类存储实例方法的定义。

## 数据结构

### Class 的结构
```objc
struct objc_class {
    Class isa;                     // 指向元类
    Class superclass;             // 指向父类
    cache_t cache;                // 方法缓存
    class_data_bits_t bits;       // 类的具体信息
    
    class_rw_t *data() {         // 类的读写数据
        return bits.data();
    }
    // ...
};

struct class_rw_t {
    uint32_t flags;
    uint32_t version;
    const class_ro_t *ro;         // 类的只读数据
    method_list_t *methods;       // 方法列表
    property_list_t *properties;  // 属性列表
    protocol_list_t *protocols;   // 协议列表
    // ...
};
```

## 类对象与元类对象

### 1. 对象的isa指针
- 实例对象的isa指向类对象
- 类对象的isa指向元类对象
- 元类对象的isa指向根元类对象
- 根元类对象的isa指向自己

### 2. 继承关系
```objc
// 类的继承关系
Student -> Person -> NSObject

// 元类的继承关系
Student's MetaClass -> Person's MetaClass -> NSObject's MetaClass -> NSObject
```

## 方法调用流程

### 1. 实例方法调用
```objc
// 实例方法调用过程
1. 通过实例的isa找到类对象
2. 在类对象的方法列表中查找
3. 找不到则通过superclass指针找父类
4. 以此类推直到NSObject
```

### 2. 类方法调用
```objc
// 类方法调用过程
1. 通过类对象的isa找到元类对象
2. 在元类对象的方法列表中查找
3. 找不到则通过superclass指针找父类的元类
4. 以此类推直到根元类
```

## 实践案例

### 1. 获取类信息
```objc
@implementation RuntimeHelper

+ (void)printClassInfo:(Class)cls {
    NSLog(@"Class: %@", cls);
    NSLog(@"Super Class: %@", class_getSuperclass(cls));
    NSLog(@"Meta Class: %@", object_getClass(cls));
    NSLog(@"Meta Super Class: %@", class_getSuperclass(object_getClass(cls)));
}

@end
```

### 2. 动态创建类
```objc
Class newClass = objc_allocateClassPair([NSObject class], "MyClass", 0);
class_addMethod(newClass, @selector(hello), (IMP)hello, "v@:");
class_addIvar(newClass, "_name", sizeof(NSString *), log2(sizeof(NSString *)), "@");
objc_registerClassPair(newClass);
```

## 注意事项

1. **方法查找**
   - 实例方法在类对象中查找
   - 类方法在元类对象中查找
   - 注意继承链的查找顺序

2. **内存管理**
   - 类对象和元类对象在程序运行期间一直存在
   - 不要随意修改类的内部结构

3. **动态创建**
   - 动态创建的类要注意内存管理
   - 需要正确设置继承关系

## 面试要点

1. 什么是类对象和元类对象？
- 类对象是一个对象,存储了类的实例方法、属性、协议等信息
- 元类对象也是一个对象,存储了类方法等类相关的信息
- 每个类在内存中都有且只有一个类对象和一个元类对象

2. 类对象和元类对象的区别是什么？
- 类对象存储实例方法列表,元类对象存储类方法列表
- 类对象可以创建实例,元类对象不能创建实例
- 类对象的isa指向元类对象,元类对象的isa指向根元类对象
- 类对象的superclass指向父类的类对象,元类对象的superclass指向父类的元类对象

3. 实例方法和类方法的调用流程是怎样的？
- 实例方法:通过实例对象的isa找到类对象,在方法列表中查找,找不到则通过superclass找父类
- 类方法:通过类对象的isa找到元类对象,在方法列表中查找,找不到则通过superclass找父元类

4. isa指针和superclass指针的作用是什么？
- isa指针用于在对象和类/元类之间建立联系,形成消息传递通道
- superclass指针用于实现继承体系,建立类/元类的继承链条

5. 如何理解Objective-C的消息发送机制？
- OC中的方法调用都是消息发送
- 通过isa指针找到对应的类/元类对象
- 在方法列表中查找对应的方法实现
- 找不到则沿继承链查找
- 最后进入消息转发流程

6. 类对象和元类对象的内存管理是怎样的？
- 类对象和元类对象在程序运行期间一直存在
- 由runtime管理其生命周期
- 不需要手动管理内存
- 不会被释放

7. 如何动态创建类？
- 使用objc_allocateClassPair创建类
- 添加成员变量和方法
- 使用objc_registerClassPair注册类
- 注意内存管理和继承关系的设置

## 相关资源

- [Objective-C Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html)
- [Understanding the Objective-C Runtime](http://cocoasamurai.blogspot.com/2010/01/understanding-objective-c-runtime.html)
- [What is a meta-class in Objective-C?](https://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html) 