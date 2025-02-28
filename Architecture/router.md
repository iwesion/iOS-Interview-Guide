# 路由设计

## 基本概念

路由(Router)是一种页面导航和模块通信的解决方案，通过 URL 的方式实现页面跳转和参数传递，使模块之间解耦。主要包括：
- URL 路由：通过 URL 进行页面跳转
- 服务路由：通过协议实现服务调用
- 参数路由：处理页面间参数传递

## 核心设计

### 1. URL 路由
```objc
// 路由协议
@protocol RouterProtocol <NSObject>
- (BOOL)canHandleURL:(NSURL *)URL;
- (void)handleURL:(NSURL *)URL parameters:(NSDictionary *)parameters;
@end

// 路由注册
@interface Router : NSObject

+ (instancetype)sharedInstance;
- (void)registerHandler:(id<RouterProtocol>)handler forScheme:(NSString *)scheme;
- (void)openURL:(NSURL *)URL parameters:(NSDictionary *)parameters;

@end
```

### 2. 服务路由
```objc
// 服务协议
@protocol ServiceRouterProtocol <NSObject>
- (id)serviceForProtocol:(Protocol *)protocol;
- (void)registerService:(id)service forProtocol:(Protocol *)protocol;
@end

// 服务实现
@interface ServiceRouter : NSObject <ServiceRouterProtocol>
@property (nonatomic, strong) NSMutableDictionary *serviceMap;
@end
```

## 实现方案

### 1. URL 解析
```objc
@implementation URLParser

- (NSDictionary *)parseURL:(NSURL *)URL {
    // 解析 scheme
    NSString *scheme = URL.scheme;
    
    // 解析 path
    NSString *path = URL.path;
    
    // 解析参数
    NSMutableDictionary *params = [NSMutableDictionary dictionary];
    NSURLComponents *components = [[NSURLComponents alloc] initWithURL:URL 
                                                           resolvingAgainstBaseURL:NO];
    for (NSURLQueryItem *item in components.queryItems) {
        params[item.name] = item.value;
    }
    
    return @{
        @"scheme": scheme,
        @"path": path,
        @"params": params
    };
}

@end
```

### 2. 路由注册
```objc
@implementation Router

- (void)registerHandler:(id<RouterProtocol>)handler forScheme:(NSString *)scheme {
    if (!handler || !scheme) return;
    
    @synchronized (self) {
        self.handlers[scheme] = handler;
    }
}

- (void)openURL:(NSURL *)URL parameters:(NSDictionary *)parameters {
    NSString *scheme = URL.scheme;
    id<RouterProtocol> handler = self.handlers[scheme];
    
    if ([handler canHandleURL:URL]) {
        [handler handleURL:URL parameters:parameters];
    }
}

@end
```

## 实践案例

### 1. 页面跳转
```objc
// 注册路由
@implementation UserRouter

- (BOOL)canHandleURL:(NSURL *)URL {
    return [URL.scheme isEqualToString:@"user"];
}

- (void)handleURL:(NSURL *)URL parameters:(NSDictionary *)parameters {
    if ([URL.path isEqualToString:@"/profile"]) {
        [self showUserProfile:parameters[@"userID"]];
    }
}

@end

// 使用路由
[[Router sharedInstance] openURL:[NSURL URLWithString:@"user://profile?userID=123"]
                    parameters:@{@"animated": @YES}];
```

### 2. 服务调用
```objc
// 注册服务
[[ServiceRouter sharedInstance] registerService:[[UserService alloc] init]
                                   forProtocol:@protocol(UserServiceProtocol)];

// 获取服务
id<UserServiceProtocol> userService = [[ServiceRouter sharedInstance] 
    serviceForProtocol:@protocol(UserServiceProtocol)];
```

## 注意事项

1. **URL 设计**
   - 统一命名规范
   - 参数类型约定
   - 版本控制

2. **安全处理**
   - URL 合法性校验
   - 参数安全检查
   - 防止注入攻击

3. **性能优化**
   - 路由表缓存
   - 懒加载处理
   - 异步处理

## 面试要点

1. 路由的主要作用是什么？
- 解耦组件间依赖
- 统一页面跳转方式
- 支持动态跳转和配置
- 实现组件化/模块化
- 支持H5和Native混合开发

2. 如何设计路由系统？
- 定义统一的URL Schema
- 实现URL解析和参数提取
- 设计路由注册和查找机制
- 提供同步/异步处理能力
- 支持拦截器和中间件
- 实现降级和容错机制

3. URL 路由和服务路由的区别？
URL路由:
- 基于URL进行跳转
- 适合页面导航
- 支持外部调用
- 参数通过URL传递

服务路由:
- 基于接口进行调用
- 适合组件间通信
- 仅支持内部调用
- 支持复杂参数传递

4. 如何处理路由安全问题？
- URL白名单校验
- 参数类型安全检查
- 权限验证
- 防止注入攻击
- 敏感数据加密
- 调用来源验证

5. 路由性能如何优化？
- 路由表缓存
- 懒加载目标页面
- 预加载常用页面
- 异步处理耗时操作
- 减少参数序列化
- 优化查找算法

6. 如何设计路由降级方案？
- 本地配置降级规则
- 远程动态下发配置
- 提供默认页面
- 错误重试机制
- 降级日志记录
- 监控和报警

7. 路由系统如何测试？
- 单元测试路由解析
- 集成测试页面跳转
- 异常场景测试
- 性能压力测试
- 安全漏洞测试
- 兼容性测试

## 相关资源

- [JLRoutes](https://github.com/joeldev/JLRoutes)
- [CTMediator](https://github.com/casatwy/CTMediator)
- [URLNavigator](https://github.com/devxoul/URLNavigator) 