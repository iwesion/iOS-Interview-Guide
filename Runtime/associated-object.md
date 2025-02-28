# Associated Object (关联对象)

## 基本概念

关联对象（Associated Objects）是 Objective-C Runtime 提供的一种为已有类添加存储属性的机制，常用于在分类中添加属性。

## 核心API

```objc
// 设置关联对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);

// 获取关联对象
id objc_getAssociatedObject(id object, const void *key);

// 移除所有关联对象
void objc_removeAssociatedObjects(id object);
```

### 关联策略（AssociationPolicy）

```objc
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           // 弱引用
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 强引用,非原子性
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   // 复制,非原子性
    OBJC_ASSOCIATION_RETAIN = 01401,       // 强引用,原子性
    OBJC_ASSOCIATION_COPY = 01403          // 复制,原子性
};
```

## 实现原理

### 1. 数据结构
```objc
// 关联对象管理结构
struct AssociationsManager {
    static AssociationsHashMap *_map;
    static pthread_mutex_t _mutex;
};

// 关联对象表
class AssociationsHashMap : public unordered_map<disguised_ptr_t, ObjectAssociationMap> {};

// 对象的关联对象表
class ObjectAssociationMap : public std::map<void *, ObjcAssociation> {};

// 关联对象
class ObjcAssociation {
    uintptr_t _policy;  // 关联策略
    id _value;          // 关联值
};
```

### 2. 存储过程
1. 获取 AssociationsManager 的全局锁
2. 获取 AssociationsHashMap
3. 根据对象地址获取 ObjectAssociationMap
4. 将键值对存储到 ObjectAssociationMap 中

## 最佳实践

### 1. 在分类中添加属性
```objc
@interface UIView (Border)
@property (nonatomic, assign) CGFloat borderWidth;
@end

@implementation UIView (Border)

- (void)setBorderWidth:(CGFloat)borderWidth {
    objc_setAssociatedObject(self, 
                            @selector(borderWidth), 
                            @(borderWidth), 
                            OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    self.layer.borderWidth = borderWidth;
}

- (CGFloat)borderWidth {
    return [objc_getAssociatedObject(self, @selector(borderWidth)) floatValue];
}

@end
```

### 2. 使用静态变量作为关联键
```objc
static char kAssociatedObjectKey;

- (void)setAssociatedObject:(id)object {
    objc_setAssociatedObject(self, 
                            &kAssociatedObjectKey, 
                            object, 
                            OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

### 3. 使用属性地址作为关联键
```objc
- (void)setName:(NSString *)name {
    objc_setAssociatedObject(self, 
                            @selector(name), 
                            name, 
                            OBJC_ASSOCIATION_COPY_NONATOMIC);
}
```

## 实践案例

### 1. 为UIView添加手势回调
```objc
@interface UIView (TapBlock)
@property (nonatomic, copy) void (^tapBlock)(void);
@end

@implementation UIView (TapBlock)

- (void)setTapBlock:(void (^)(void))tapBlock {
    objc_setAssociatedObject(self, @selector(tapBlock), tapBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
    
    if (tapBlock) {
        UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleTap)];
        [self addGestureRecognizer:tap];
    }
}

- (void (^)(void))tapBlock {
    return objc_getAssociatedObject(self, @selector(tapBlock));
}

- (void)handleTap {
    if (self.tapBlock) {
        self.tapBlock();
    }
}

@end
```

### 2. 运行时存储额外数据
```objc
@interface NSObject (Runtime)
@property (nonatomic, strong) id extraData;
@end

@implementation NSObject (Runtime)

- (void)setExtraData:(id)extraData {
    objc_setAssociatedObject(self, @selector(extraData), extraData, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)extraData {
    return objc_getAssociatedObject(self, @selector(extraData));
}

@end
```

## 注意事项

1. **内存管理**
   - 选择正确的关联策略
   - 注意循环引用问题
   - 对象释放时关联对象会自动释放

2. **性能影响**
   - 关联对象的读写比实例变量慢
   - 适量使用关联对象

3. **线程安全**
   - 关联对象操作是线程安全的
   - 但需要注意自定义 getter/setter 的线程安全

4. **调试难度**
   - 关联对象不会显示在调试器中
   - 建议添加调试辅助方法

## 面试要点

1. 关联对象的实现原理是什么？
- Runtime维护了一个全局的AssociationsManager
- 通过AssociationsHashMap存储所有对象的关联对象
- 每个对象有一个ObjectAssociationMap存储该对象的所有关联对象
- key是关联对象的key,value是ObjcAssociation(存储关联策略和关联值)

2. 关联对象的内存管理策略有哪些？
- OBJC_ASSOCIATION_ASSIGN: 弱引用,不改变引用计数
- OBJC_ASSOCIATION_RETAIN_NONATOMIC: 强引用,非原子性
- OBJC_ASSOCIATION_COPY_NONATOMIC: 复制,非原子性  
- OBJC_ASSOCIATION_RETAIN: 强引用,原子性
- OBJC_ASSOCIATION_COPY: 复制,原子性

3. 关联对象是如何实现线程安全的？
- AssociationsManager中使用pthread_mutex_t互斥锁
- 所有关联对象的操作都是线程安全的
- 但自定义的getter/setter方法需要自行保证线程安全

4. 什么场景下使用关联对象？
- 为分类添加属性
- 为已有类添加额外存储
- 为对象添加运行时数据
- 实现自定义功能扩展

5. 关联对象的性能如何？
- 比实例变量的访问慢
- 涉及哈希表查找和锁操作
- 不适合频繁访问的场景
- 建议适量使用

6. 关联对象的内存管理是怎样的？
- 对象释放时自动清理关联对象
- 根据关联策略决定引用方式
- 需要注意避免循环引用
- 可以手动调用objc_removeAssociatedObjects清理

7. 如何安全地使用关联对象？
- 选择合适的关联策略
- 使用静态变量或selector作为key
- 注意内存管理和循环引用
- 确保getter/setter线程安全
- 适量使用,避免滥用

## 相关资源

- [Associated Objects](https://nshipster.com/associated-objects/)
- [Objective-C Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html)
- [Associated Objects in iOS](https://www.cocoanetics.com/2012/06/associated-objects/) 