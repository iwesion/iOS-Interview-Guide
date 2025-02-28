# 组件化架构

## 基本概念

组件化是将 App 按照功能模块拆分成相对独立的组件，每个组件都可以独立开发、测试和复用。主要包括：
- 基础组件：工具类、基础UI等
- 业务组件：具体业务功能
- 中间件：组件间通信、路由等

## 核心设计

### 1. 组件协议
```objc
// 组件生命周期协议
@protocol ComponentProtocol <NSObject>

@required
- (void)setup;
- (void)teardown;

@optional
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;

@end

// 组件注册
@interface ComponentManager : NSObject

+ (instancetype)sharedInstance;
- (void)registerComponent:(id<ComponentProtocol>)component;
- (void)setupAllComponents;

@end
```

### 2. 路由设计
```objc
// 路由协议
@protocol RouterProtocol <NSObject>

- (BOOL)canHandleURL:(NSURL *)URL;
- (void)handleURL:(NSURL *)URL parameters:(NSDictionary *)parameters;

@end

// 路由实现
@interface Router : NSObject

+ (instancetype)sharedInstance;
- (void)registerHandler:(id<RouterProtocol>)handler forScheme:(NSString *)scheme;
- (void)openURL:(NSURL *)URL parameters:(NSDictionary *)parameters;

@end
```

## 实现方案

### 1. 组件管理
```objc
@implementation ComponentManager

- (void)setupAllComponents {
    for (id<ComponentProtocol> component in self.components) {
        if ([component respondsToSelector:@selector(setup)]) {
            [component setup];
        }
    }
}

- (void)handleApplicationDidFinishLaunching:(NSDictionary *)launchOptions {
    for (id<ComponentProtocol> component in self.components) {
        if ([component respondsToSelector:@selector(application:didFinishLaunchingWithOptions:)]) {
            [component application:[UIApplication sharedApplication] 
         didFinishLaunchingWithOptions:launchOptions];
        }
    }
}

@end
```

### 2. 服务注册
```objc
// 服务协议
@protocol UserServiceProtocol <NSObject>
- (User *)currentUser;
- (void)loginWithCompletion:(void(^)(BOOL success))completion;
@end

// 服务注册
@interface ServiceManager : NSObject

+ (instancetype)sharedInstance;
- (void)registerService:(id)service protocol:(Protocol *)protocol;
- (id)serviceForProtocol:(Protocol *)protocol;

@end
```

## 实践案例

### 1. 基础组件
```objc
// 网络组件
@interface NetworkComponent : NSObject <ComponentProtocol>

@property (nonatomic, strong) NetworkConfig *config;

- (void)setupWithConfig:(NetworkConfig *)config;
- (void)request:(NSString *)url params:(NSDictionary *)params completion:(void(^)(id response))completion;

@end

// 存储组件
@interface StorageComponent : NSObject <ComponentProtocol>

- (void)saveObject:(id)object forKey:(NSString *)key;
- (id)objectForKey:(NSString *)key;

@end
```

### 2. 业务组件
```objc
// 用户组件
@interface UserComponent : NSObject <ComponentProtocol, UserServiceProtocol>

@property (nonatomic, strong) User *currentUser;

- (void)login:(NSString *)username password:(NSString *)password completion:(void(^)(BOOL))completion;
- (void)logout;

@end

// 支付组件
@interface PaymentComponent : NSObject <ComponentProtocol, PaymentServiceProtocol>

- (void)payWithOrder:(Order *)order completion:(void(^)(BOOL))completion;

@end
```

## 通信机制

### 1. URL 路由
```objc
// 注册路由
[[Router sharedInstance] registerHandler:self forScheme:@"user"];

// 处理路由
- (void)handleURL:(NSURL *)URL parameters:(NSDictionary *)parameters {
    if ([URL.path isEqualToString:@"/profile"]) {
        [self showUserProfile:parameters[@"userID"]];
    }
}

// 调用路由
[[Router sharedInstance] openURL:[NSURL URLWithString:@"user://profile?userID=123"] 
                    parameters:@{@"animated": @YES}];
```

### 2. 服务调用
```objc
// 注册服务
[[ServiceManager sharedInstance] registerService:[[UserComponent alloc] init] 
                                     protocol:@protocol(UserServiceProtocol)];

// 使用服务
id<UserServiceProtocol> userService = [[ServiceManager sharedInstance] 
    serviceForProtocol:@protocol(UserServiceProtocol)];
[userService loginWithCompletion:^(BOOL success) {
    // 处理登录结果
}];
```

## 优缺点

### 优点
1. 高度解耦
2. 可独立开发
3. 便于维护
4. 代码复用
5. 支持动态化

### 缺点
1. 架构复杂
2. 开发成本高
3. 调试困难
4. 需要统一规范

## 注意事项

1. **组件设计**
   - 合理划分粒度
   - 降低耦合度
   - 接口设计要稳定

2. **通信机制**
   - 统一路由规则
   - 服务接口要清晰
   - 处理循环依赖

3. **版本管理**
   - 组件版本控制
   - 接口向后兼容
   - 依赖管理

## 面试要点

1. 组件化的主要目的是什么？
- 降低代码耦合度
- 提高代码复用性
- 支持并行开发
- 便于维护和扩展
- 实现动态化部署

2. 如何设计组件间通信？
- 基于协议的服务注册和发现
- URL路由跳转
- 消息总线或通知
- 依赖注入
- 本地中间件

3. 组件化架构的优缺点？
优点:
- 高度解耦,独立开发
- 代码复用性强
- 便于维护和扩展
- 支持动态部署
- 提高开发效率

缺点:
- 架构设计复杂
- 前期开发成本高
- 调试和测试困难
- 需要完善的规范

4. 如何解决组件间循环依赖？
- 引入中间层解耦
- 使用依赖注入
- 接口下沉
- 重新梳理业务边界
- 采用事件驱动

5. 组件化的版本管理策略？
- 语义化版本号
- 组件发布规范
- 依赖管理工具
- 接口兼容性测试
- 版本更新日志

6. 如何进行组件化改造？
- 业务梳理和模块划分
- 制定统一规范
- 设计通信机制
- 循序渐进重构
- 完善测试体系

7. 组件化测试怎么做？
- 单元测试覆盖核心逻辑
- 组件集成测试
- 端到端测试
- 接口兼容性测试
- 性能和内存测试

## 相关资源

- [iOS Modular Architecture](https://github.com/casatwy/CTMediator)
- [BeeHive](https://github.com/alibaba/BeeHive)
- [组件化架构设计](https://casatwy.com/modulization_in_action.html) 