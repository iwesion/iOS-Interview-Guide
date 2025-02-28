# MVC 架构模式

## 基本概念

MVC(Model-View-Controller)是iOS开发中最基础的架构模式，它将应用程序分为三个核心组件：
- Model：数据模型层，负责业务逻辑和数据处理
- View：视图层，负责界面展示
- Controller：控制器层，负责业务逻辑和界面的衔接

## 组件职责

### 1. Model
```objc
// 数据模型
@interface User : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, copy) NSString *avatar;

- (void)updateUserInfo:(NSDictionary *)info;
- (void)saveToLocal;

@end

// 业务逻辑
@interface UserManager : NSObject

- (void)loginWithUsername:(NSString *)username 
                password:(NSString *)password 
              completion:(void(^)(User *user, NSError *error))completion;

@end
```

### 2. View
```objc
@interface UserProfileView : UIView

@property (nonatomic, strong) UIImageView *avatarView;
@property (nonatomic, strong) UILabel *nameLabel;
@property (nonatomic, strong) UILabel *ageLabel;

- (void)updateWithUser:(User *)user;

@end

@implementation UserProfileView

- (void)updateWithUser:(User *)user {
    self.nameLabel.text = user.name;
    self.ageLabel.text = @(user.age).stringValue;
    // 更新头像
}

@end
```

### 3. Controller
```objc
@interface UserProfileViewController : UIViewController

@property (nonatomic, strong) UserProfileView *profileView;
@property (nonatomic, strong) UserManager *userManager;
@property (nonatomic, strong) User *user;

@end

@implementation UserProfileViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupViews];
    [self loadData];
}

- (void)loadData {
    [self.userManager loginWithUsername:@"test" password:@"123" completion:^(User *user, NSError *error) {
        if (user) {
            self.user = user;
            [self.profileView updateWithUser:user];
        }
    }];
}

@end
```

## 通信方式

### 1. Controller -> Model
```objc
// 直接调用
[self.userManager loginWithUsername:username password:password completion:^(User *user, NSError *error) {
    // 处理结果
}];
```

### 2. Model -> Controller
```objc
// 通过回调
- (void)loginWithUsername:(NSString *)username 
                password:(NSString *)password 
              completion:(void(^)(User *user, NSError *error))completion {
    // 异步操作
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 处理登录
        dispatch_async(dispatch_get_main_queue(), ^{
            completion(user, nil);
        });
    });
}
```

### 3. View -> Controller
```objc
// 通过代理
@protocol UserProfileViewDelegate <NSObject>
- (void)profileViewDidTapAvatar:(UserProfileView *)view;
@end

@interface UserProfileView : UIView
@property (nonatomic, weak) id<UserProfileViewDelegate> delegate;
@end

// 通过事件响应
- (void)avatarTapped {
    if ([self.delegate respondsToSelector:@selector(profileViewDidTapAvatar:)]) {
        [self.delegate profileViewDidTapAvatar:self];
    }
}
```

## 实践案例

### 1. 列表页面
```objc
// Model
@interface ListItem : NSObject
@property (nonatomic, copy) NSString *title;
@property (nonatomic, copy) NSString *detail;
@end

// View
@interface ListCell : UITableViewCell
- (void)updateWithItem:(ListItem *)item;
@end

// Controller
@interface ListViewController : UIViewController <UITableViewDelegate, UITableViewDataSource>
@property (nonatomic, strong) NSArray<ListItem *> *items;
@property (nonatomic, strong) UITableView *tableView;
@end
```

### 2. 表单页面
```objc
// Model
@interface FormData : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *email;
- (BOOL)validate;
@end

// View
@interface FormView : UIView
@property (nonatomic, strong) UITextField *nameField;
@property (nonatomic, strong) UITextField *emailField;
@end

// Controller
@interface FormViewController : UIViewController
@property (nonatomic, strong) FormView *formView;
@property (nonatomic, strong) FormData *formData;
@end
```

## 优缺点

### 优点
1. 结构清晰，易于理解
2. 各组件职责明确
3. 适合小型项目
4. 开发效率高

### 缺点
1. Controller 容易臃肿
2. 组件耦合度较高
3. 难以进行单元测试
4. 代码复用性差

## 注意事项

1. **避免 Controller 臃肿**
   - 抽取公共方法
   - 使用 Category
   - 将复杂逻辑移至 Model

2. **降低耦合度**
   - 使用代理模式
   - 使用通知中心
   - 避免直接引用

3. **提高可测试性**
   - 抽取业务逻辑
   - 使用依赖注入
   - 避免硬编码

## 面试要点

1. MVC 各个组件的职责是什么？
Model:
- 负责数据的存储和处理
- 包含业务逻辑
- 数据持久化
- 网络请求

View:
- 负责界面展示
- 处理用户交互
- 将事件传递给Controller
- 不包含业务逻辑

Controller:
- 协调Model和View
- 处理业务流程
- 响应View事件
- 更新Model和View

2. 如何解决 Controller 臃肿问题？
- 抽取公共逻辑到基类
- 使用 Category 扩展功能
- 将业务逻辑移至 Model 层
- 使用 MVVM 或其他架构模式
- 创建 Manager 类处理特定功能
- 使用组合而非继承

3. MVC 各组件之间如何通信？
Model -> Controller:
- KVO
- 通知
- Block回调
- 代理

View -> Controller:
- Target-Action
- 代理
- Block回调
- 事件响应链

Controller -> Model:
- 直接方法调用

Controller -> View:
- 直接方法调用
- 数据绑定

4. 为什么说 MVC 难以测试？
- Controller 承担过多职责
- 组件间耦合度高
- 依赖关系难以模拟
- UI 测试困难
- 业务逻辑分散

5. 如何改进传统 MVC？
- 使用依赖注入
- 引入 Service 层
- 采用 MVVM 架构
- 使用响应式编程
- 增加 Router 层
- 实现组件化

6. MVC 和其他架构模式的区别？
MVVM:
- 引入 ViewModel
- 数据绑定
- 更好的可测试性
- 更低的耦合度

MVP:
- Presenter 替代 Controller
- 严格的数据流向
- View 更加被动
- 更容易测试

VIPER:
- 职责划分更细
- 更好的可维护性
- 适合大型项目
- 学习成本高

7. 什么场景适合使用 MVC？
- 小型项目
- 界面简单
- 业务逻辑不复杂
- 团队熟悉 MVC
- 快速开发
- 原型验证

## 相关资源

- [Model-View-Controller](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html)
- [Cocoa Core Competencies](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Cocoa.html)
- [iOS App Programming Guide](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html) 