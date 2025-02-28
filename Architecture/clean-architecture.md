# Clean Architecture

## 基本概念

Clean Architecture 是一种分层架构模式，它通过明确的依赖规则将应用程序分为不同的层次，使得代码更加清晰、可测试和可维护。主要分为：
- Entities：业务规则和实体
- Use Cases：应用程序特定的业务规则
- Interface Adapters：转换数据格式
- Frameworks & Drivers：外部框架和工具

## 核心原则

### 1. 依赖规则
```objc
// 内层不依赖外层
@interface User : NSObject  // Entity
@property (nonatomic, copy) NSString *userID;
@property (nonatomic, copy) NSString *name;
@end

@protocol UserRepository <NSObject>  // Interface Adapter
- (void)fetchUserWithID:(NSString *)userID completion:(void(^)(User *))completion;
@end

@interface FetchUserUseCase : NSObject  // Use Case
- (void)executeWithUserID:(NSString *)userID completion:(void(^)(User *))completion;
@end
```

### 2. 接口隔离
```objc
// 使用协议定义接口
@protocol UserDataSource <NSObject>
- (void)fetchUserWithID:(NSString *)userID completion:(void(^)(User *))completion;
@end

@interface APIUserDataSource : NSObject <UserDataSource>
@end

@interface LocalUserDataSource : NSObject <UserDataSource>
@end
```

## 实现示例

### 1. Entities
```objc
@interface User : NSObject

@property (nonatomic, copy) NSString *userID;
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *email;

- (BOOL)validate;

@end

@interface UserValidator : NSObject

- (BOOL)validateUser:(User *)user;

@end
```

### 2. Use Cases
```objc
@interface FetchUserUseCase : NSObject

@property (nonatomic, strong) id<UserRepository> repository;

- (instancetype)initWithRepository:(id<UserRepository>)repository;
- (void)executeWithUserID:(NSString *)userID 
               completion:(void(^)(User *, NSError *))completion;

@end

@implementation FetchUserUseCase

- (void)executeWithUserID:(NSString *)userID 
               completion:(void(^)(User *, NSError *))completion {
    [self.repository fetchUserWithID:userID completion:^(User *user) {
        if ([user validate]) {
            completion(user, nil);
        } else {
            completion(nil, [NSError errorWithDomain:@"Invalid user" code:-1 userInfo:nil]);
        }
    }];
}

@end
```

### 3. Interface Adapters
```objc
@interface UserPresenter : NSObject

- (void)presentUser:(User *)user;
- (UserViewModel *)viewModelFromUser:(User *)user;

@end

@interface UserRepository : NSObject <UserRepository>

@property (nonatomic, strong) id<UserDataSource> remoteDataSource;
@property (nonatomic, strong) id<UserDataSource> localDataSource;

- (void)fetchUserWithID:(NSString *)userID completion:(void(^)(User *))completion;

@end
```

### 4. Frameworks & Drivers
```objc
@interface APIClient : NSObject

- (void)fetchUserWithID:(NSString *)userID completion:(void(^)(NSDictionary *))completion;

@end

@interface DatabaseManager : NSObject

- (void)saveUser:(User *)user;
- (User *)fetchUserWithID:(NSString *)userID;

@end
```

## 实践案例

### 1. 用户管理模块
```objc
// Use Case
@interface UserManagementUseCase : NSObject

- (void)registerUser:(User *)user completion:(void(^)(BOOL))completion;
- (void)updateUser:(User *)user completion:(void(^)(BOOL))completion;
- (void)deleteUser:(NSString *)userID completion:(void(^)(BOOL))completion;

@end

// Repository
@interface UserManagementRepository : NSObject

@property (nonatomic, strong) APIClient *apiClient;
@property (nonatomic, strong) DatabaseManager *database;

@end

// Presenter
@interface UserManagementPresenter : NSObject

- (void)presentSuccess:(NSString *)message;
- (void)presentError:(NSError *)error;

@end
```

### 2. 认证模块
```objc
// Use Case
@interface AuthenticationUseCase : NSObject

- (void)loginWithUsername:(NSString *)username 
                password:(NSString *)password 
              completion:(void(^)(User *, NSError *))completion;

- (void)logout:(void(^)(BOOL))completion;

@end

// Repository
@interface AuthenticationRepository : NSObject

@property (nonatomic, strong) id<AuthenticationDataSource> dataSource;
@property (nonatomic, strong) id<TokenStorage> tokenStorage;

@end
```

## 优缺点

### 优点
1. 高度可测试
2. 独立于框架
3. 独立于UI
4. 独立于数据库
5. 业务规则隔离

### 缺点
1. 前期开发成本高
2. 可能过度设计
3. 学习曲线陡峭
4. 小项目可能不适合

## 注意事项

1. **依赖规则**
   - 依赖只能指向内层
   - 内层不知道外层的存在
   - 使用依赖注入

2. **边界划分**
   - 明确每层的职责
   - 避免跨层访问
   - 合理使用接口

3. **测试策略**
   - 单元测试覆盖核心业务
   - 集成测试验证交互
   - 端到端测试确保功能

## 面试要点

1. Clean Architecture 的核心原则是什么？
- 依赖规则:内层不依赖外层
- 关注点分离:每层职责单一
- 接口隔离:通过接口定义交互
- 可测试性:业务逻辑易于测试
- 独立于框架:核心逻辑不依赖具体实现

2. 各层之间如何通信？
- 通过接口和协议定义交互方式
- 使用回调或闭包传递数据
- 遵循依赖倒置原则
- 采用数据传输对象(DTO)转换
- 避免跨层直接访问

3. 如何处理依赖注入？
- 构造函数注入
- 属性注入
- 接口注入
- 使用依赖注入容器
- 遵循控制反转原则

4. 如何进行测试？
- 单元测试:测试业务规则
- 集成测试:测试层间交互
- 端到端测试:测试完整流程
- Mock 外部依赖
- 使用测试替身(Test Double)

5. 与其他架构模式的区别？
- 比 MVC 职责划分更细
- 比 MVVM 更注重业务规则
- 比 VIPER 更灵活
- 更强调依赖规则
- 更适合大型项目

6. 适用于什么场景？
- 大型复杂项目
- 需要长期维护的项目
- 团队规模较大
- 对可测试性要求高
- 业务规则复杂的项目

7. 如何处理跨层数据传递？
- 使用数据映射器
- 定义数据传输对象
- 遵循单向数据流
- 避免直接传递实体对象
- 合理使用依赖注入

## 相关资源

- [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Clean Architecture for iOS](https://tech.olx.com/clean-architecture-for-ios-development-4c0740b57a01)
- [iOS Clean Architecture](https://github.com/sergdort/CleanArchitectureRxSwift) 