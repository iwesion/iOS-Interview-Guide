# 模块解耦

## 基本概念

模块解耦是指降低系统各个模块之间的依赖关系，提高代码的可维护性和可复用性。主要方法包括：
- 依赖注入：通过外部注入依赖
- 协议解耦：通过协议定义接口
- 中间件：通过中间层隔离模块

## 解耦方案

### 1. 依赖注入
```objc
// 依赖注入协议
@protocol NetworkService <NSObject>
- (void)request:(NSString *)url completion:(void(^)(id response))completion;
@end

// 使用依赖注入
@interface UserManager : NSObject
- (instancetype)initWithNetworkService:(id<NetworkService>)service;
@property (nonatomic, strong) id<NetworkService> networkService;
@end

@implementation UserManager
- (void)fetchUserInfo {
    [self.networkService request:@"/user/info" completion:^(id response) {
        // 处理响应
    }];
}
@end
```

### 2. 协议解耦
```objc
// 定义服务协议
@protocol UserServiceProtocol <NSObject>
- (void)login:(NSString *)username password:(NSString *)password;
- (User *)currentUser;
@end

// 服务实现
@interface UserService : NSObject <UserServiceProtocol>
@end

// 使用服务
@interface LoginViewController : UIViewController
@property (nonatomic, strong) id<UserServiceProtocol> userService;
@end
```

## 实现方式

### 1. 面向接口编程
```objc
// 定义接口
@protocol DataPersistence <NSObject>
- (void)saveData:(id)data forKey:(NSString *)key;
- (id)dataForKey:(NSString *)key;
@end

// 实现类
@interface UserDefaultsPersistence : NSObject <DataPersistence>
@end

@interface CoreDataPersistence : NSObject <DataPersistence>
@end

// 使用者
@interface DataManager : NSObject
- (instancetype)initWithPersistence:(id<DataPersistence>)persistence;
@end
```

### 2. 中间件模式
```objc
// 中间件协议
@protocol Middleware <NSObject>
- (void)handle:(Request *)request completion:(void(^)(Response *))completion;
@end

// 日志中间件
@interface LogMiddleware : NSObject <Middleware>
@end

// 缓存中间件
@interface CacheMiddleware : NSObject <Middleware>
@end

// 中间件管理
@interface MiddlewareManager : NSObject
- (void)addMiddleware:(id<Middleware>)middleware;
- (void)handleRequest:(Request *)request;
@end
```

## 实践案例

### 1. 业务模块解耦
```objc
// 模块协议
@protocol LoginModule <NSObject>
- (void)showLoginView;
- (BOOL)isLoggedIn;
@end

@protocol PaymentModule <NSObject>
- (void)startPayment:(Order *)order;
@end

// 模块注册
@interface ModuleManager : NSObject
+ (void)registerModule:(id)module forProtocol:(Protocol *)protocol;
+ (id)moduleForProtocol:(Protocol *)protocol;
@end
```

### 2. 组件通信
```objc
// 事件总线
@interface EventBus : NSObject
+ (void)postEvent:(NSString *)event data:(id)data;
+ (void)subscribe:(NSString *)event handler:(void(^)(id data))handler;
@end

// 使用示例
[EventBus subscribe:@"UserDidLogin" handler:^(User *user) {
    // 处理登录事件
}];

[EventBus postEvent:@"UserDidLogin" data:user];
```

## 注意事项

1. **接口设计**
   - 保持接口稳定
   - 职责单一
   - 避免过度抽象

2. **依赖管理**
   - 控制依赖方向
   - 避免循环依赖
   - 合理分层

3. **性能考虑**
   - 避免过度解耦
   - 控制中间层数量
   - 注意内存开销

## 面试要点

1. 为什么需要模块解耦？
- 降低模块间耦合度
- 提高代码可维护性
- 支持并行开发
- 便于单元测试
- 提高代码复用性

2. 常用的解耦方式有哪些？
- 依赖注入
- 协议/接口隔离
- 中间件/中介者模式
- 事件总线
- 路由机制

3. 依赖注入的实现方式？
- 构造函数注入
- 属性注入
- 方法注入
- 接口注入
- 使用依赖注入容器

4. 如何处理模块间通信？
- 基于协议的服务注册发现
- 事件总线/通知
- URL路由
- 回调/闭包
- 依赖注入

5. 解耦和性能的平衡？
- 避免过多中间层
- 合理控制抽象层次
- 关键路径直接调用
- 合理使用缓存
- 延迟加载非核心模块

6. 如何避免过度解耦？
- 合理划分模块边界
- 保持适度抽象
- 遵循YAGNI原则
- 关注实际业务需求
- 权衡开发成本和收益

7. 模块解耦的最佳实践？
- 制定统一的接口规范
- 合理使用设计模式
- 建立完善的文档
- 做好版本管理
- 编写单元测试

## 相关资源

- [Dependency Injection](https://www.objc.io/issues/15-testing/dependency-injection/)
- [Protocol-Oriented Programming](https://developer.apple.com/videos/play/wwdc2015/408/)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) 