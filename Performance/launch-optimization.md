# 启动优化

## 基本概念

App 启动时间是用户体验的重要指标。启动过程分为三个阶段：
1. pre-main：系统加载动态库、rebase/binding、ObjC setup等
2. main()：main函数到首页渲染完成
3. 首页渲染：首页视图控制器加载到可交互

## 启动时间测量

### 1. 系统日志
```objc
// 在 main.m 中添加
CFAbsoluteTime StartTime;

int main(int argc, char * argv[]) {
    StartTime = CFAbsoluteTimeGetCurrent();
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

// 在首页记录
CFAbsoluteTime LinkTime = CFAbsoluteTimeGetCurrent();
NSLog(@"启动时间: %f", (LinkTime - StartTime));
```

### 2. 使用 Instruments
```objc
// 使用 Time Profiler 工具
// 分析 pre-main 阶段
DYLD_PRINT_STATISTICS=1
DYLD_PRINT_STATISTICS_DETAILS=1
```

## 优化方案

### 1. pre-main 阶段优化
```objc
// 1. 减少动态库
// 合并功能类似的动态库
// 使用静态库

// 2. 减少类和方法数量
// 删除无用代码
// 延迟加载不必要的类

// 3. 减少 load 方法
+ (void)initialize {
    // 使用 initialize 替代 load
}
```

### 2. main() 阶段优化
```objc
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 1. 异步初始化 SDK
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [self initThirdPartySDK];
    });
    
    // 2. 延迟加载不必要的服务
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [self initOptionalServices];
    });
    
    return YES;
}

@end
```

### 3. 首页优化
```objc
@implementation HomeViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 1. 视图懒加载
    if (!_tableView) {
        _tableView = [[UITableView alloc] init];
    }
    
    // 2. 数据预加载
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [self prefetchData];
    });
}

@end
```

## 实践案例

### 1. 二进制重排
```objc
// 1. 添加编译标记
__attribute__((section("__TEXT,__objc_methname")))

// 2. 生成 order 文件
// 使用 LinkMap 分析符号顺序
```

### 2. 启动器模式
```objc
@interface AppLauncher : NSObject

+ (instancetype)sharedInstance;
- (void)registerService:(id<LaunchService>)service;
- (void)loadServices;

@end

@implementation AppLauncher

- (void)loadServices {
    // 按优先级顺序加载服务
    for (id<LaunchService> service in self.services) {
        if (service.async) {
            dispatch_async(dispatch_get_global_queue(0, 0), ^{
                [service start];
            });
        } else {
            [service start];
        }
    }
}

@end
```

### 3. 闪屏优化
```objc
// 1. 使用纯色背景
<key>UILaunchScreen</key>
<dict>
    <key>UIColorName</key>
    <string>launchScreenBackground</string>
</dict>

// 2. 预加载首页数据
- (void)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 在闪屏期间预加载数据
    [self prefetchHomeData];
}
```

## 性能监控

### 1. 启动时间监控
```objc
@interface LaunchProfiler : NSObject

+ (void)beginRecord;
+ (void)recordPoint:(NSString *)point;
+ (void)endRecord;

@end

@implementation LaunchProfiler

+ (void)recordPoint:(NSString *)point {
    CFAbsoluteTime current = CFAbsoluteTimeGetCurrent();
    NSLog(@"%@ time: %f", point, current - StartTime);
}

@end
```

### 2. 卡顿监控
```objc
@interface ANRDetector : NSObject

+ (void)startMonitoring;
+ (void)stopMonitoring;

@end

@implementation ANRDetector

+ (void)startMonitoring {
    // 使用 runloop 监控主线程卡顿
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(
        kCFAllocatorDefault,
        kCFRunLoopAllActivities,
        YES,
        0,
        ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
            [self handleRunLoopActivity:activity];
        });
    CFRunLoopAddObserver(CFRunLoopGetMain(), observer, kCFRunLoopCommonModes);
}

@end
```

## 注意事项

1. **优化原则**
   - 只加载必要的内容
   - 能延迟的都延迟加载
   - 能异步的都异步处理

2. **监控指标**
   - 启动时间不超过3秒
   - 首页可交互时间不超过5秒
   - 定期检查启动性能

3. **测试验证**
   - 在不同设备上测试
   - 考虑网络环境影响
   - 进行线上监控

## 面试要点

1. App 的启动过程是怎样的？
- pre-main阶段:加载动态库、rebase/binding、ObjC setup等
- main()阶段:main函数到首页渲染完成
- 首页渲染阶段:首页视图控制器加载到可交互

2. 如何测量启动时间？
- 在main函数记录启动时间点
- 在首页记录结束时间点
- 使用DYLD_PRINT_STATISTICS查看pre-main耗时
- 使用Instruments的Time Profiler分析
- 添加自定义打点统计

3. 启动优化的主要方向有哪些？
- 减少动态库数量和大小
- 减少类和方法数量
- 减少load方法的使用
- 延迟加载不必要的内容
- 优化首页渲染流程
- 合理安排启动任务

4. 如何处理启动时的任务调度？
- 区分必要和非必要任务
- 将非必要任务延迟执行
- 使用异步并发处理合适的任务
- 设置任务优先级
- 监控任务执行时间

5. 二进制重排的原理是什么？
- 分析启动时的函数调用顺序
- 将启动需要的函数排列到一起
- 减少page fault,提高缓存命中率
- 通过order file指定符号顺序
- 减少启动时的IO操作

6. 如何监控启动性能？
- 记录关键节点的时间戳
- 监控CPU、内存使用情况
- 统计启动时间分布
- 设置性能监控报警
- 进行线上数据分析

7. 有哪些常见的启动优化方案？
- 合并相似功能的动态库
- 删除无用的类和方法
- 将initialize替代load方法
- 使用懒加载和延迟加载
- 异步处理合适的任务
- 优化资源加载方式
- 实现二进制重排优化

## 相关资源

- [App Launch Time: Past, Present, and Future](https://developer.apple.com/videos/play/wwdc2017/413/)
- [Optimizing App Launch](https://developer.apple.com/documentation/xcode/improving-your-app-s-performance/optimizing-app-launch)
- [iOS App Launch Performance Optimization](https://www.objc.io/issues/19-debugging/launch-time-performance/) 