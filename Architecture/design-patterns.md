# 设计模式

## 基本概念

设计模式是软件开发中常见问题的最佳实践解决方案。在 iOS 开发中常用的设计模式包括：
- 创建型模式：单例、工厂、构建者等
- 结构型模式：适配器、装饰器、代理等
- 行为型模式：观察者、命令、策略等

## 常用模式

### 1. 单例模式
```objc
// 线程安全的单例
@implementation SharedManager

+ (instancetype)sharedInstance {
    static SharedManager *instance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[SharedManager alloc] init];
    });
    return instance;
}

// 防止外部创建实例
- (instancetype)init {
    @throw [NSException exceptionWithName:@"Singleton"
                                 reason:@"Use +sharedInstance instead"
                               userInfo:nil];
    return nil;
}

@end
```

### 2. 代理模式
```objc
// 代理协议
@protocol DataSourceDelegate <NSObject>
- (void)dataDidUpdate:(NSArray *)data;
- (void)didFailWithError:(NSError *)error;
@end

// 使用代理
@interface DataManager : NSObject
@property (nonatomic, weak) id<DataSourceDelegate> delegate;
- (void)fetchData;
@end

@implementation DataManager
- (void)fetchData {
    // 数据获取成功后
    [self.delegate dataDidUpdate:data];
}
@end
```

### 3. 观察者模式
```objc
// KVO
@interface UserViewModel : NSObject
@property (nonatomic, copy) NSString *username;
@end

// 添加观察者
[self.viewModel addObserver:self
                forKeyPath:@"username"
                   options:NSKeyValueObservingOptionNew
                   context:nil];

// 通知
[[NSNotificationCenter defaultCenter] addObserver:self
                                      selector:@selector(handleNotification:)
                                          name:@"UserDidUpdateNotification"
                                        object:nil];
```

### 4. 工厂模式
```objc
// 抽象工厂
@protocol ViewControllerFactory <NSObject>
- (UIViewController *)createViewController;
@end

// 具体工厂
@interface ProfileViewControllerFactory : NSObject <ViewControllerFactory>
@end

@implementation ProfileViewControllerFactory
- (UIViewController *)createViewController {
    ProfileViewController *vc = [[ProfileViewController alloc] init];
    // 配置 ViewController
    return vc;
}
@end
```

## 实践案例

### 1. 网络请求封装
```objc
// 策略模式 + 工厂模式
@protocol RequestStrategy <NSObject>
- (void)executeRequest:(Request *)request completion:(void(^)(Response *))completion;
@end

@interface NetworkManager : NSObject
+ (instancetype)sharedInstance;
- (void)setStrategy:(id<RequestStrategy>)strategy;
- (void)request:(Request *)request completion:(void(^)(Response *))completion;
@end
```

### 2. 界面构建
```objc
// 构建者模式
@interface AlertViewBuilder : NSObject
- (AlertViewBuilder *)withTitle:(NSString *)title;
- (AlertViewBuilder *)withMessage:(NSString *)message;
- (AlertViewBuilder *)addAction:(NSString *)title handler:(void(^)(void))handler;
- (UIAlertController *)build;
@end

// 使用
AlertViewBuilder *builder = [[AlertViewBuilder alloc] init];
UIAlertController *alert = [[[[builder withTitle:@"提示"]
                                    withMessage:@"确认删除？"]
                                    addAction:@"确定" handler:^{ }]
                                    build];
```

## 注意事项

1. **选择合适的模式**
   - 考虑实际需求
   - 避免过度设计
   - 保持代码简洁

2. **正确使用模式**
   - 理解模式本质
   - 遵循设计原则
   - 注意性能影响

3. **模式组合**
   - 灵活组合使用
   - 解决复杂问题
   - 保持可维护性

## 面试要点

1. iOS 中常用的设计模式有哪些？
- 单例模式:NSUserDefaults、UIApplication
- 代理模式:UITableViewDelegate、UICollectionViewDelegate 
- 观察者模式:通知中心、KVO
- 工厂模式:UIButton buttonWithType:
- 构建者模式:链式调用创建UI
- 策略模式:网络请求策略、缓存策略
- 命令模式:撤销重做、队列任务
- 装饰器模式:Category、Extension
- 适配器模式:兼容不同接口

2. 单例模式的实现要点？
- dispatch_once 保证线程安全
- 重写 allocWithZone 防止外部分配内存
- 实现 copyWithZone 返回单例对象
- 使用 static 变量保存实例
- 考虑内存管理和释放时机

3. 代理和通知的区别？
代理:
- 一对一通信
- 强耦合但类型安全
- 同步调用
- 适合UI交互场景
通知:
- 一对多通信
- 松耦合但类型不安全
- 异步调用
- 适合跨模块通信

4. 如何选择合适的设计模式？
- 分析实际业务需求
- 考虑扩展性和维护性
- 权衡性能和复杂度
- 遵循最小原则
- 参考成熟框架实践

5. 设计模式的使用原则？
- 单一职责原则
- 开闭原则
- 里氏替换原则
- 接口隔离原则
- 依赖倒置原则
- 最少知识原则

6. 如何避免设计模式的滥用？
- 从实际需求出发
- 避免过度抽象
- 保持代码简洁
- 权衡开发成本
- 考虑团队接受度

7. 设计模式在框架设计中的应用？
- 网络层:策略模式、工厂模式
- UI框架:代理模式、构建者模式
- 数据存储:单例模式、装饰器模式
- 消息传递:观察者模式、命令模式
- 业务模块:适配器模式、中介者模式

## 相关资源

- [Design Patterns in Swift](https://github.com/ochococo/Design-Patterns-In-Swift)
- [iOS Design Patterns](https://www.raywenderlich.com/477-design-patterns-on-ios-using-swift-part-1-2)
- [Cocoa Design Patterns](https://developer.apple.com/documentation/swift/cocoa_design_patterns) 