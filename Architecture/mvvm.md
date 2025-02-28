# MVVM 架构模式

## 基本概念

MVVM(Model-View-ViewModel)是对 MVC 的改进，通过引入 ViewModel 层来解决 Controller 臃肿的问题。它将展示逻辑从 Controller 中抽离出来，使得代码更加清晰和易于测试。

## 核心组件

### 1. Model
```objc
@interface User : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, copy) NSString *avatar;

@end

@interface UserService : NSObject

- (void)fetchUserWithID:(NSString *)userID 
             completion:(void(^)(User *user, NSError *error))completion;

@end
```

### 2. View
```objc
@interface UserProfileView : UIView

@property (nonatomic, strong) UIImageView *avatarView;
@property (nonatomic, strong) UILabel *nameLabel;
@property (nonatomic, strong) UILabel *ageLabel;

@end

@interface UserProfileViewController : UIViewController

@property (nonatomic, strong) UserProfileView *profileView;
@property (nonatomic, strong) UserProfileViewModel *viewModel;

@end
```

### 3. ViewModel
```objc
@interface UserProfileViewModel : NSObject

@property (nonatomic, strong) User *user;
@property (nonatomic, copy) NSString *displayName;
@property (nonatomic, copy) NSString *displayAge;
@property (nonatomic, copy) void (^updateUI)(void);

- (void)fetchUserData;
- (void)updateUserProfile;

@end

@implementation UserProfileViewModel

- (void)setUser:(User *)user {
    _user = user;
    self.displayName = user.name;
    self.displayAge = [NSString stringWithFormat:@"%ld岁", (long)user.age];
    if (self.updateUI) {
        self.updateUI();
    }
}

@end
```

## 数据绑定

### 1. KVO 绑定
```objc
@implementation UserProfileViewController

- (void)setupBinding {
    [self.viewModel addObserver:self 
                    forKeyPath:@"displayName" 
                       options:NSKeyValueObservingOptionNew 
                       context:nil];
}

- (void)observeValueForKeyPath:(NSString *)keyPath 
                     ofObject:(id)object 
                       change:(NSDictionary *)change 
                      context:(void *)context {
    if ([keyPath isEqualToString:@"displayName"]) {
        self.profileView.nameLabel.text = self.viewModel.displayName;
    }
}

@end
```

### 2. RAC 绑定
```objc
@implementation UserProfileViewController

- (void)setupBinding {
    RAC(self.profileView.nameLabel, text) = RACObserve(self.viewModel, displayName);
    RAC(self.profileView.ageLabel, text) = RACObserve(self.viewModel, displayAge);
    
    @weakify(self);
    [[self.profileView.updateButton rac_signalForControlEvents:UIControlEventTouchUpInside] 
        subscribeNext:^(id x) {
        @strongify(self);
        [self.viewModel updateUserProfile];
    }];
}

@end
```

## 实践案例

### 1. 列表页面
```objc
// ViewModel
@interface ListViewModel : NSObject

@property (nonatomic, strong) NSArray<ListItemViewModel *> *items;
@property (nonatomic, assign) BOOL isLoading;
@property (nonatomic, copy) void (^updateUI)(void);

- (void)fetchListData;
- (void)loadMore;

@end

// ViewController
@implementation ListViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    @weakify(self);
    self.viewModel.updateUI = ^{
        @strongify(self);
        [self.tableView reloadData];
    };
    
    [self.viewModel fetchListData];
}

@end
```

### 2. 表单页面
```objc
// ViewModel
@interface FormViewModel : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *email;
@property (nonatomic, assign) BOOL isValid;
@property (nonatomic, copy) void (^updateUI)(void);

- (void)validate;
- (void)submit;

@end

// ViewController
@implementation FormViewController

- (void)setupBinding {
    RAC(self.submitButton, enabled) = RACObserve(self.viewModel, isValid);
    
    [[self.submitButton rac_signalForControlEvents:UIControlEventTouchUpInside] 
        subscribeNext:^(id x) {
        [self.viewModel submit];
    }];
}

@end
```

## 优缺点

### 优点
1. 展示逻辑从 Controller 中解耦
2. 更容易进行单元测试
3. 数据绑定使代码更简洁
4. ViewModel 可以复用

### 缺点
1. 简单页面使用成本高
2. 数据绑定增加学习成本
3. 调试相对困难
4. 内存占用可能增加

## 注意事项

1. **合理划分职责**
   - ViewModel 只处理展示逻辑
   - Controller 只处理用户交互
   - 避免 ViewModel 过度臃肿

2. **选择合适的绑定方式**
   - KVO 适合简单场景
   - RAC 适合复杂场景
   - 注意内存管理

3. **性能优化**
   - 避免过度绑定
   - 合理使用 RAC
   - 注意循环引用

## 面试要点

1. MVVM 相比 MVC 有什么优势？
   - 更好的代码分离,展示逻辑从Controller中解耦
   - 更容易进行单元测试,ViewModel可独立测试
   - 数据绑定使代码更简洁,减少样板代码
   - ViewModel可以复用,提高代码复用性

2. ViewModel 的职责是什么？
   - 处理视图展示相关的业务逻辑
   - 数据格式转换和验证
   - 为View提供数据
   - 不直接持有View,通过数据绑定通信

3. 如何实现数据绑定？
   - KVO方式:适合简单场景
   - RAC框架:适合复杂场景
   - 通知中心:适合一对多场景
   - 代理模式:适合一对一场景

4. MVVM 如何进行单元测试？
   - ViewModel可独立测试,不依赖UI
   - 测试数据转换逻辑
   - 测试业务规则验证
   - 测试状态变化

5. 什么场景适合使用 MVVM？
   - 复杂的UI交互逻辑
   - 需要频繁更新UI的场景
   - 需要代码复用的场景
   - 需要单元测试的场景

6. MVVM 的内存管理注意事项？
   - 注意循环引用
   - 合理使用weak/strong
   - RAC信号订阅的内存释放
   - 及时移除通知和KVO观察

7. 如何处理 MVVM 的模块通信？
   - 通知中心
   - 代理模式
   - RAC信号
   - 依赖注入
   - 路由机制

## 相关资源

- [Introduction to MVVM](https://www.objc.io/issues/13-architecture/mvvm/)
- [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
- [MVVM Tutorial](https://www.raywenderlich.com/34-design-patterns-by-tutorials-mvvm) 