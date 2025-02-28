# VIPER 架构模式

## 基本概念

VIPER 是一种基于 Clean Architecture 的 iOS 应用架构模式，它将应用程序分为五个主要组件：
- View：负责界面展示
- Interactor：包含业务逻辑
- Presenter：协调 View 和 Interactor
- Entity：数据模型
- Router：处理导航逻辑

## 核心组件

### 1. View
```objc
@protocol UserProfileViewProtocol <NSObject>
- (void)showUserProfile:(UserProfileViewModel *)viewModel;
- (void)showLoading;
- (void)hideLoading;
- (void)showError:(NSString *)message;
@end

@interface UserProfileViewController : UIViewController <UserProfileViewProtocol>
@property (nonatomic, strong) id<UserProfilePresenterProtocol> presenter;
@end
```

### 2. Interactor
```objc
@protocol UserProfileInteractorProtocol <NSObject>
- (void)fetchUserProfile:(NSString *)userID;
- (void)updateUserProfile:(User *)user;
@end

@interface UserProfileInteractor : NSObject <UserProfileInteractorProtocol>
@property (nonatomic, weak) id<UserProfilePresenterProtocol> presenter;
@property (nonatomic, strong) id<UserProfileService> service;
@end
```

### 3. Presenter
```objc
@protocol UserProfilePresenterProtocol <NSObject>
- (void)viewDidLoad;
- (void)didFetchUserProfile:(User *)user;
- (void)didUpdateUserProfile:(BOOL)success;
@end

@interface UserProfilePresenter : NSObject <UserProfilePresenterProtocol>
@property (nonatomic, weak) id<UserProfileViewProtocol> view;
@property (nonatomic, strong) id<UserProfileInteractorProtocol> interactor;
@property (nonatomic, strong) id<UserProfileRouterProtocol> router;
@end
```

### 4. Entity
```objc
@interface User : NSObject
@property (nonatomic, copy) NSString *userID;
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *email;
@end

@interface UserProfileViewModel : NSObject
@property (nonatomic, copy) NSString *displayName;
@property (nonatomic, copy) NSString *displayEmail;
@end
```

### 5. Router
```objc
@protocol UserProfileRouterProtocol <NSObject>
- (void)navigateToEditProfile;
- (void)navigateToSettings;
@end

@interface UserProfileRouter : NSObject <UserProfileRouterProtocol>
@property (nonatomic, weak) UIViewController *viewController;
+ (UIViewController *)createModule;
@end
```

## 模块通信

### 1. 组件间通信
```objc
// Presenter -> View
- (void)didFetchUserProfile:(User *)user {
    UserProfileViewModel *viewModel = [self viewModelFromUser:user];
    [self.view showUserProfile:viewModel];
}

// View -> Presenter
- (void)editButtonTapped {
    [self.presenter didTapEditButton];
}

// Presenter -> Interactor
- (void)viewDidLoad {
    [self.interactor fetchUserProfile:self.userID];
}

// Interactor -> Presenter
- (void)userProfileFetched:(User *)user {
    [self.presenter didFetchUserProfile:user];
}
```

### 2. 模块间通信
```objc
// Router 处理模块间跳转
@implementation UserProfileRouter

+ (UIViewController *)createModule {
    UserProfileViewController *view = [[UserProfileViewController alloc] init];
    UserProfilePresenter *presenter = [[UserProfilePresenter alloc] init];
    UserProfileInteractor *interactor = [[UserProfileInteractor alloc] init];
    UserProfileRouter *router = [[UserProfileRouter alloc] init];
    
    view.presenter = presenter;
    presenter.view = view;
    presenter.interactor = interactor;
    presenter.router = router;
    interactor.presenter = presenter;
    router.viewController = view;
    
    return view;
}

@end
```

## 实践案例

### 1. 用户资料模块
```objc
// Module Builder
@interface UserProfileBuilder : NSObject

+ (UIViewController *)buildUserProfileModule;

@end

@implementation UserProfileBuilder

+ (UIViewController *)buildUserProfileModule {
    // 创建并配置所有组件
    UserProfileViewController *view = [[UserProfileViewController alloc] init];
    UserProfilePresenter *presenter = [[UserProfilePresenter alloc] init];
    UserProfileInteractor *interactor = [[UserProfileInteractor alloc] init];
    UserProfileRouter *router = [[UserProfileRouter alloc] init];
    
    // 建立依赖关系
    [self setupDependencies:view
                 presenter:presenter
                interactor:interactor
                   router:router];
    
    return view;
}

@end
```

### 2. 列表模块
```objc
// Protocols
@protocol ListViewProtocol <NSObject>
- (void)showItems:(NSArray<ItemViewModel *> *)items;
- (void)showLoading;
- (void)hideLoading;
@end

@protocol ListPresenterProtocol <NSObject>
- (void)viewDidLoad;
- (void)didSelectItemAtIndex:(NSInteger)index;
@end

@protocol ListInteractorProtocol <NSObject>
- (void)fetchItems;
@end

@protocol ListRouterProtocol <NSObject>
- (void)navigateToDetail:(Item *)item;
@end
```

## 优缺点

### 优点
1. 职责分离清晰
2. 高度可测试
3. 可维护性好
4. 适合大型项目

### 缺点
1. 代码量大
2. 学习成本高
3. 小项目过度设计
4. 配置复杂

## 注意事项

1. **模块划分**
   - 合理划分模块边界
   - 避免组件间直接通信
   - 保持单一职责

2. **依赖管理**
   - 使用依赖注入
   - 避免循环依赖
   - 合理使用协议

3. **测试策略**
   - 每个组件可单独测试
   - 使用 Mock 对象
   - 注重接口设计

## 面试要点

1. VIPER 的各个组件职责是什么？
- View: 负责界面展示和用户交互
- Interactor: 包含业务逻辑,数据处理
- Presenter: 协调View和Interactor,处理展示逻辑
- Entity: 数据模型,业务实体
- Router: 处理页面导航,模块跳转

2. 如何处理组件间的通信？
- 通过协议定义接口
- 依赖注入实现解耦
- 单向数据流
- 避免组件间直接引用

3. 与 MVC/MVVM 相比有什么优势？
- 职责划分更清晰
- 更容易进行单元测试
- 更适合大型项目
- 更好的可维护性
- 更容易进行团队协作

4. 如何进行单元测试？
- 每个组件可独立测试
- 使用协议便于Mock
- 测试展示逻辑
- 测试业务逻辑
- 测试导航逻辑

5. 适合什么样的项目？
- 大型复杂项目
- 需要严格分层的项目
- 多人协作的项目
- 需要高度可测试性的项目
- 长期维护的项目

6. 如何处理模块间的通信？
- 使用Router进行导航
- 使用Service层解耦
- 使用通知中心
- 使用依赖注入
- 使用事件总线

7. 如何避免配置的复杂性？
- 使用模板自动生成
- 抽象公共基类
- 统一命名规范
- 合理拆分模块
- 文档规范化

## 相关资源

- [VIPER Architecture](https://www.objc.io/issues/13-architecture/viper/)
- [iOS Architecture Patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52)
- [Clean Architecture + VIPER](https://github.com/sergdort/CleanArchitectureRxSwift) 