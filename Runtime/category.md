# Category 实现原理

## 基本概念

Category（分类）是 Objective-C 的一个重要特性，它允许我们在不修改原有类的源代码的情况下，动态地为已有类添加新的方法。

## 底层数据结构

Category 在 Runtime 中的数据结构如下：

```objc
struct category_t {
    const char *name;                 // 分类名
    classref_t cls;                   // 类的引用
    struct method_list_t *instanceMethods;  // 实例方法列表
    struct method_list_t *classMethods;     // 类方法列表
    struct protocol_list_t *protocols;      // 协议列表
    struct property_list_t *instanceProperties;  // 实例属性列表
    struct property_list_t *_classProperties;    // 类属性列表
};
```

## 加载过程

### 1. 编译期
- 编译器将 Category 中的方法、属性、协议等信息编码到 category_t 结构体中
- 将这些信息保存在可执行文件的 __DATA 段中

### 2. 运行时
```objc
// Runtime 加载流程
1. 通过 objc_init 初始化运行时
2. _objc_init 调用 map_images
3. map_images 调用 map_images_nolock
4. map_images_nolock 调用 read_images
5. read_images 中调用 remethodizeClass 进行方法合并
```

### 3. 方法合并过程
```objc
static void attachCategories(Class cls, const locstamped_category_t *cats_list, uint32_t cats_count, int flags) {
    if (!cats_count) return;
    
    method_list_t **mlists = (method_list_t **)malloc(cats_count * sizeof(*mlists));
    property_list_t **proplists = (property_list_t **)malloc(cats_count * sizeof(*proplists));
    protocol_list_t **protolists = (protocol_list_t **)malloc(cats_count * sizeof(*protolists));

    // 将所有分类的方法、属性、协议整理到数组中
    // 后编译的分类优先级更高，会被放在数组前面
    
    // 将整理好的列表插入到类的方法、属性、协议列表前面
    prepareMethodLists(cls, mlists, mcount, NO, fromBundle, baseMethods);
    preparePropertyLists(cls, proplists, propcount, fromBundle, baseProperties);
    prepareProtocolLists(cls, protolists, protocount, fromBundle, baseProtocols);
}
```

## 特性解析

### 1. 方法覆盖
- Category 的方法会"覆盖"原类方法
- 多个 Category 中相同方法，后编译的生效
- 原类方法仍然存在，但是被放到了方法列表的后面

### 2. 加载顺序
```objc
// 编译顺序决定加载顺序
// 可在 Build Phases -> Compile Sources 中查看和调整编译顺序
```

### 3. 关联对象
```objc
@interface NSObject (Extension)
@property (nonatomic, copy) NSString *name;
@end

@implementation NSObject (Extension)
- (void)setName:(NSString *)name {
    objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)name {
    return objc_getAssociatedObject(self, @selector(name));
}
@end
```

## 最佳实践

### 1. 命名规范
```objc
// 使用前缀避免命名冲突
@interface UIViewController (XXXTracking)
- (void)xxx_trackEvent:(NSString *)event;
@end
```

### 2. 方法冲突处理
```objc
// 运行时检查方法是否存在
if (![self respondsToSelector:@selector(originalMethod)]) {
    // 添加方法实现
}
```

### 3. 分类整理
```objc
// 按功能分类
UIViewController+Tracking.h
UIViewController+Network.h
UIViewController+Utils.h
```

## 实践案例

### 1. 为系统类添加功能
```objc
@interface UIImage (Resize)

- (UIImage *)xxx_resizeToSize:(CGSize)size;
- (UIImage *)xxx_roundCornerWithRadius:(CGFloat)radius;

@end
```

### 2. 模块化功能扩展
```objc
// 网络模块
@interface NSObject (Networking)
- (void)sendRequest:(NSURLRequest *)request;
@end

// 持久化模块
@interface NSObject (Storage)
- (void)saveToFile:(NSString *)path;
@end
```

## 注意事项

1. **方法覆盖**
   - 注意分类方法对原类方法的影响
   - 合理管理多个分类的编译顺序

2. **性能影响**
   - 适量使用分类
   - 注意方法查找的性能开销

3. **维护性**
   - 合理组织分类文件
   - 做好文档注释

4. **限制**
   - 不能添加实例变量
   - 不能添加属性存储值（需使用关联对象）

## 面试要点

1. Category 的实现原理是什么？
   - Category 是通过 Runtime 在运行时将分类的方法、属性、协议等添加到原类中
   - 底层结构是 category_t，包含方法列表、属性列表、协议列表等
   - 在程序运行时，Runtime 会将分类的数据合并到类对象中

2. Category 和 Extension 的区别是什么？
   - Extension 是编译时决议，必须有原类的源码
   - Category 是运行时决议，不需要原类源码
   - Extension 可以添加实例变量，Category 不能
   - Extension 中声明的方法必须实现，Category 可以不实现
   - Extension 一般用来隐藏类的私有信息，Category 用来扩展类的功能

3. Category 能否添加实例变量？为什么？
   - 不能直接添加实例变量
   - 因为编译时类的内存布局已经确定，运行时无法修改
   - 可以使用关联对象（Associated Object）来模拟实例变量

4. Category 中的方法是如何被加载的？
   - 在 Runtime 初始化时，通过 _objc_init 开始加载
   - 通过 read_images 读取镜像文件
   - 调用 remethodizeClass: 进行方法合并
   - 将分类的方法列表添加到类对象的方法列表中

5. 为什么 Category 的方法优先级比原类方法高？
   - Category 的方法被附加到方法列表的前面
   - 方法查找按照方法列表的顺序查找
   - 找到方法后就不再继续查找
   - 所以分类方法会优先于原类方法被调用

6. Category 的加载顺序是怎样的？
   - 按照编译顺序决定
   - 后编译的分类优先级更高
   - 可以通过修改 Build Phases 中的编译顺序调整
   - 同名方法会相互覆盖，最后编译的生效

7. 关联对象的原理是什么？
   - [关联对象的原理](Runtime/Category/category-test.md)
   - [关联对象的原理](Runtime/Category/test2.md)
   - 通过 Runtime 的关联对象 API 实现
   - 使用全局的 AssociationsManager 管理
   - 通过 disguised_ptr_t 作为 key 建立关联
   - 线程安全由 AssociationsManager 的锁机制保证
   - 对象释放时，关联对象也会自动释放

## 相关资源

- [Apple Category Documentation](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)
- [Objective-C Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html)
- [Associated Objects](https://nshipster.com/associated-objects/)
