# 热修复方案

## 基本概念

热修复是在不发版的情况下修复线上 Bug 的技术方案。主要包括：
- JSPatch：通过 JavaScript 修改 OC 运行时
- 动态下发：远程更新业务逻辑
- 代码注入：运行时替换方法实现
- 安全性：防止恶意代码注入

## 实现方案

### 1. JSPatch 方案
```objc
// JSPatch 配置
@implementation JPEngine

+ (void)startEngine {
    // 初始化 JSPatch 引擎
    [JPEngine startEngine];
    
    // 加载补丁脚本
    NSString *script = [NSString stringWithContentsOfFile:@"patch.js" 
                                               encoding:NSUTF8StringEncoding 
                                                  error:nil];
    [JPEngine evaluateScript:script];
}

// 注册自定义方法
+ (void)defineMethod {
    [JPEngine defineStruct:@{
        @"name": @"CGRect",
        @"types": @"{{CGPoint}{CGSize}}"
    }];
}

@end
```

```javascript
// patch.js
defineClass('MyViewController', {
    viewDidLoad: function() {
        self.ORIGviewDidLoad();  // 调用原方法
        
        // 修复代码
        var label = UILabel.alloc().init();
        label.setText("Fixed!");
        self.view().addSubview(label);
    }
});
```

### 2. 动态部署
```objc
// 动态更新管理
@interface HotfixManager : NSObject

@property (nonatomic, strong) NSString *currentVersion;
@property (nonatomic, copy) void (^updateCallback)(BOOL success);

// 检查更新
- (void)checkUpdate {
    NSURL *url = [NSURL URLWithString:@"https://api.example.com/hotfix"];
    NSURLSession *session = [NSURLSession sharedSession];
    
    [[session dataTaskWithURL:url completionHandler:^(NSData *data, 
                                                    NSURLResponse *response, 
                                                    NSError *error) {
        if (error) {
            self.updateCallback(NO);
            return;
        }
        
        // 解析补丁信息
        NSDictionary *patch = [NSJSONSerialization JSONObjectWithData:data 
                                                            options:0 
                                                              error:nil];
        // 下载并应用补丁
        [self applyPatch:patch];
        
    }] resume];
}

// 应用补丁
- (void)applyPatch:(NSDictionary *)patch {
    NSString *script = patch[@"script"];
    if (script) {
        [self executePatch:script];
        self.updateCallback(YES);
    }
}

@end
```

### 3. Method Swizzling
```objc
// 方法替换
@implementation MethodSwizzling

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        
        // 获取原方法和新方法
        SEL originalSelector = @selector(originalMethod);
        SEL swizzledSelector = @selector(swizzledMethod);
        
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
        // 交换方法实现
        method_exchangeImplementations(originalMethod, swizzledMethod);
    });
}

// 新的方法实现
- (void)swizzledMethod {
    // 修复后的实现
    NSLog(@"Fixed implementation");
}

@end
```

## 安全机制

### 1. 代码签名
```objc
// 验证补丁签名
@implementation SecurityCheck

- (BOOL)verifyPatch:(NSData *)patchData withSignature:(NSString *)signature {
    // 验证签名
    SecKeyRef publicKey = [self loadPublicKey];
    if (!publicKey) return NO;
    
    SecTransformRef verifier = SecVerifyTransformCreate(publicKey, 
                                                       (__bridge CFDataRef)patchData,
                                                       NULL);
    
    CFDataRef sig = (__bridge CFDataRef)[signature dataUsingEncoding:NSUTF8StringEncoding];
    SecTransformSetAttribute(verifier,
                           kSecTransformInputAttributeName,
                           sig,
                           NULL);
    
    CFErrorRef error;
    CFTypeRef result = SecTransformExecute(verifier, &error);
    
    return result == kCFBooleanTrue;
}

@end
```

### 2. 权限控制
```objc
// 权限管理
@implementation PermissionControl

- (BOOL)checkPermission:(NSString *)patchId {
    // 检查补丁权限
    NSDictionary *permissions = @{
        @"network": @YES,
        @"fileAccess": @NO,
        @"userDefaults": @YES
    };
    
    return [permissions[patchId] boolValue];
}

// 沙箱隔离
- (NSString *)sandboxPathForPatch:(NSString *)patchId {
    NSString *documentsPath = NSSearchPathForDirectoriesInDomains(
        NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
    return [documentsPath stringByAppendingPathComponent:patchId];
}

@end
```

## 注意事项

1. **安全性**
   - 代码签名验证
   - 权限控制
   - 防止注入攻击

2. **稳定性**
   - 补丁版本管理
   - 回滚机制
   - 异常处理

3. **合规性**
   - 遵守 App Store 规则
   - 避免违规操作
   - 保护用户隐私

## 面试要点

1. 热修复的实现原理？
- 运行时方法替换
- Method Swizzling 技术
- 动态下发补丁
- 代码注入机制
- 脚本解释执行

2. JSPatch 的工作机制？
- JavaScript 引擎解析脚本
- 通过 Runtime 修改方法实现
- 方法映射和转换
- 参数类型适配
- 异常处理机制

3. 如何保证热修复安全？
- 代码签名验证
- 补丁加密传输
- 权限控制和沙箱
- 注入代码审核
- 版本号校验

4. 热修复的应用场景？
- 紧急 Bug 修复
- UI 微调优化
- 文案更新
- A/B 测试
- 特性开关

5. 版本管理策略？
- 补丁版本号管理
- 增量更新机制
- 回滚版本存储
- 补丁依赖检查
- 更新策略控制

6. 如何处理修复失败？
- 异常捕获机制
- 自动回滚策略
- 日志记录分析
- 备用方案切换
- 用户提示处理

7. 合规性考虑？
- 遵守 App Store 规则
- 避免违规操作
- 用户隐私保护
- 数据安全合规
- 审核政策适配

## 相关资源

- [JSPatch](https://github.com/bang590/JSPatch)
- [Method Swizzling](https://nshipster.com/method-swizzling/)
- [iOS Security](https://developer.apple.com/documentation/security) 