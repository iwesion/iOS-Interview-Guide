# KVO/KVC 原理

## 基本概念

KVO(Key-Value Observing) 和 KVC(Key-Value Coding) 是 Cocoa 的两个基础特性：
- KVO：对象属性变化的观察机制
- KVC：间接访问对象属性的机制
- 实现原理：动态生成子类和方法调用
- 应用场景：数据绑定和监听

## KVO 实现

### 1. 基本使用
```objc
// KVO 的使用
@implementation KVOExample

- (void)setupKVO {
    // 添加观察者
    [self.person addObserver:self
                 forKeyPath:@"name"
                    options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld
                    context:nil];
    
    // 观察者方法
    - (void)observeValueForKeyPath:(NSString *)keyPath
                          ofObject:(id)object
                            change:(NSDictionary *)change
                           context:(void *)context {
        if ([keyPath isEqualToString:@"name"]) {
            NSString *oldName = change[NSKeyValueChangeOldKey];
            NSString *newName = change[NSKeyValueChangeNewKey];
            NSLog(@"Name changed from %@ to %@", oldName, newName);
        }
    }
    
    // 移除观察者
    - (void)dealloc {
        [self.person removeObserver:self forKeyPath:@"name"];
    }
}

@end
```

### 2. 实现原理
```objc
// KVO 的底层实现
@implementation KVOImplementation

- (void)explainKVO {
    // 1. 动态生成子类
    NSString *originalClassName = NSStringFromClass([self.person class]);
    NSString *kvoClassName = [NSString stringWithFormat:@"NSKVONotifying_%@", originalClassName];
    
    // 2. 重写 setter 方法
    /*
     生成的子类大致实现：
     - (void)setName:(NSString *)name {
         [self willChangeValueForKey:@"name"];
         [super setName:name];
         [self didChangeValueForKey:@"name"];
     }
     */
    
    // 3. 修改 isa 指针
    // 将对象的 isa 指向新生成的子类
}

// 手动触发 KVO
- (void)manualKVO {
    [self willChangeValueForKey:@"name"];
    _name = @"NewName";
    [self didChangeValueForKey:@"name"];
}

@end
```

## KVC 实现

### 1. 基本使用
```objc
// KVC 的使用
@implementation KVCExample

- (void)useKVC {
    // 设置值
    [self.person setValue:@"John" forKey:@"name"];
    [self.person setValue:@25 forKeyPath:@"address.number"];
    
    // 获取值
    NSString *name = [self.person valueForKey:@"name"];
    NSNumber *number = [self.person valueForKeyPath:@"address.number"];
    
    // 集合操作
    NSArray *array = @[@1, @2, @3, @4];
    NSNumber *avg = [array valueForKeyPath:@"@avg.self"];
    NSNumber *sum = [array valueForKeyPath:@"@sum.self"];
}

@end
```

### 2. 查找顺序
```objc
// KVC 的查找规则
@implementation KVCAccessor

// 获取值的顺序
- (id)getValueForKey:(NSString *)key {
    // 1. 查找 get<Key>, <key>, is<Key>, _<key> 方法
    SEL getters[] = {
        NSSelectorFromString([NSString stringWithFormat:@"get%@", key.capitalizedString]),
        NSSelectorFromString(key),
        NSSelectorFromString([NSString stringWithFormat:@"is%@", key.capitalizedString]),
        NSSelectorFromString([NSString stringWithFormat:@"_%@", key])
    };
    
    // 2. 查找实例变量 _<key>, <key>
    Ivar ivars[] = {
        class_getInstanceVariable([self class], "_key"),
        class_getInstanceVariable([self class], "key")
    };
    
    // 3. 如果都没找到，调用 valueForUndefinedKey:
}

// 设置值的顺序
- (void)setValue:(id)value forKey:(NSString *)key {
    // 1. 查找 set<Key>: 方法
    SEL setter = NSSelectorFromString([NSString stringWithFormat:@"set%@:", key.capitalizedString]);
    
    // 2. 查找实例变量 _<key>, <key>
    // 3. 如果都没找到，调用 setValue:forUndefinedKey:
}

@end
```

## 实践应用

### 1. 数据绑定
```objc
// MVVM 数据绑定
@implementation DataBinding

- (void)bindViewModel {
    // 观察 ViewModel 属性变化
    [self.viewModel addObserver:self
                    forKeyPath:@"title"
                       options:NSKeyValueObservingOptionNew
                       context:nil];
    
    // 更新 UI
    - (void)observeValueForKeyPath:(NSString *)keyPath
                          ofObject:(id)object
                            change:(NSDictionary *)change
                           context:(void *)context {
        if ([keyPath isEqualToString:@"title"]) {
            self.titleLabel.text = change[NSKeyValueChangeNewKey];
        }
    }
}

@end
```

### 2. 属性验证
```objc
// KVC 验证
@implementation ValidationExample

// 验证值
- (BOOL)validateValue:(inout id *)ioValue forKey:(NSString *)inKey error:(out NSError **)outError {
    if ([inKey isEqualToString:@"age"]) {
        NSNumber *age = *ioValue;
        if ([age integerValue] < 0 || [age integerValue] > 120) {
            if (outError) {
                *outError = [NSError errorWithDomain:@"ValidationError"
                                              code:1
                                          userInfo:@{NSLocalizedDescriptionKey: @"Invalid age"}];
            }
            return NO;
        }
    }
    return YES;
}

@end
```

## 注意事项

1. **KVO 注意点**
   - 配对添加和移除
   - 观察者内存管理
   - 触发时机控制

2. **KVC 注意点**
   - 键值路径正确性
   - 类型安全
   - 性能影响

3. **最佳实践**
   - 合理使用自动/手动触发
   - 异常处理
   - 线程安全

## 面试要点

1. KVO 的实现原理？
- 动态生成观察对象的子类
- 重写 setter 方法添加通知
- 修改对象的 isa 指针指向新类
- 在 setter 中调用 willChangeValueForKey 和 didChangeValueForKey
- 发送观察者通知

2. KVC 的查找规则？
- 按顺序查找 getter：getKey、key、isKey、_key
- 按顺序查找 setter：setKey:、_setKey:
- 如果都没找到，调用 valueForUndefinedKey:
- 支持点语法访问嵌套属性
- 可以通过数组下标访问集合

3. 如何手动触发 KVO？
- 调用 willChangeValueForKey:
- 修改属性值
- 调用 didChangeValueForKey:
- 或直接调用 setValue:forKey:
- 需要在同一线程操作

4. KVO 和 Notification 的区别？
- KVO 针对属性变化，Notification 用于事件通知
- KVO 自动触发，Notification 需手动发送
- KVO 一对一，Notification 一对多
- KVO 需要属性支持，Notification 无限制
- KVO 耦合度较高，Notification 解耦性好

5. KVC 的性能如何？
- 比直接访问属性慢
- 涉及方法查找和消息转发
- 需要进行类型转换
- 字符串比较消耗性能
- 建议适度使用

6. 如何防止 KVO 崩溃？
- 确保配对添加和移除观察者
- 检查 keyPath 是否存在
- dealloc 中移除观察者
- 使用 try-catch 捕获异常
- 合理管理观察者生命周期

7. KVO 的使用场景？
- 数据和 UI 绑定
- 监听模型变化
- 响应式编程
- 属性依赖管理
- 调试和日志记录

## 相关资源

- [Key-Value Observing Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html)
- [Key-Value Coding Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html)
- [KVO Implementation Details](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOImplementation.html) 