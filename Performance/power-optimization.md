# 耗电优化

## 基本概念

耗电优化是指通过合理使用系统资源，减少不必要的电量消耗，延长应用运行时间。主要包括 CPU、网络、定位等资源的优化。

## 耗电来源

### 1. CPU 使用
```objc
// 过度使用 CPU
while (true) {
    // 持续计算
    [self heavyComputation];
}
```

### 2. 网络请求
```objc
// 频繁的网络请求
- (void)startPolling {
    [NSTimer scheduledTimerWithTimeInterval:1.0
                                   repeats:YES
                                     block:^(NSTimer *timer) {
        [self fetchDataFromServer];
    }];
}
```

### 3. 定位服务
```objc
// 持续使用定位
- (void)startLocationUpdates {
    self.locationManager.desiredAccuracy = kCLLocationAccuracyBest;
    [self.locationManager startUpdatingLocation];
}
```

## 优化方案

### 1. CPU 优化
```objc
// 1. 后台任务优化
- (void)applicationDidEnterBackground:(UIApplication *)application {
    UIBackgroundTaskIdentifier taskID = [application beginBackgroundTaskWithExpirationHandler:^{
        [application endBackgroundTask:taskID];
    }];
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 执行必要的后台任务
        [application endBackgroundTask:taskID];
    });
}

// 2. 使用定时器
- (void)startTimer {
    // 使用 CADisplayLink 替代 NSTimer
    self.displayLink = [CADisplayLink displayLinkWithTarget:self
                                                selector:@selector(update:)];
    [self.displayLink addToRunLoop:[NSRunLoop mainRunLoop]
                          forMode:NSRunLoopCommonModes];
}
```

### 2. 网络优化
```objc
// 1. 批量请求
- (void)batchRequest {
    // 收集请求
    if (self.pendingRequests.count < 10) {
        [self.pendingRequests addObject:request];
        return;
    }
    
    // 批量发送
    [self sendBatchRequests:self.pendingRequests];
}

// 2. 合理的请求策略
- (void)setupNetworkPolicy {
    // 使用 URLSession 的后台传输
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration backgroundSessionConfiguration:@"com.example.background"];
    config.discretionary = YES; // 系统自动选择最佳时机传输
}
```

### 3. 定位优化
```objc
// 1. 按需定位
- (void)startLocationWithAccuracy:(CLLocationAccuracy)accuracy {
    self.locationManager.desiredAccuracy = accuracy;
    self.locationManager.distanceFilter = 100; // 最小更新距离
    [self.locationManager startUpdatingLocation];
}

// 2. 区分前后台
- (void)applicationDidEnterBackground:(UIApplication *)application {
    // 降低定位精度
    self.locationManager.desiredAccuracy = kCLLocationAccuracyKilometer;
}
```

## 监控方案

### 1. Energy Log
```objc
// 使用 Xcode Energy Log
// Xcode -> Debug -> Debug Workflow -> Energy Log

// 监控关键指标
- CPU 使用率
- 网络活动
- 定位服务
- 后台任务
```

### 2. 自定义监控
```objc
@implementation PowerMonitor

+ (void)startMonitoring {
    // 监控 CPU 使用率
    [self monitorCPUUsage];
    
    // 监控网络活动
    [self monitorNetworkActivity];
    
    // 监控定位服务
    [self monitorLocationServices];
}

@end
```

## 实践案例

### 1. 后台任务优化
```objc
@implementation BackgroundTaskManager

- (void)handleBackgroundTask {
    __block UIBackgroundTaskIdentifier taskID = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        [[UIApplication sharedApplication] endBackgroundTask:taskID];
        taskID = UIBackgroundTaskInvalid;
    }];
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 执行重要的后台任务
        [self processImportantTask];
        
        [[UIApplication sharedApplication] endBackgroundTask:taskID];
        taskID = UIBackgroundTaskInvalid;
    });
}

@end
```

### 2. 网络请求优化
```objc
@implementation NetworkOptimizer

- (void)setupNetworkConfig {
    // 1. 使用合适的网络请求策略
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
    config.waitsForConnectivity = YES;
    
    // 2. 实现请求重试
    config.timeoutIntervalForResource = 30;
    config.timeoutIntervalForRequest = 30;
}

@end
```

## 注意事项

1. **资源使用**
   - 按需使用系统资源
   - 及时释放不需要的资源
   - 注意后台任务管理

2. **网络策略**
   - 合理的请求频率
   - 适当的超时时间
   - 正确的重试策略

3. **定位服务**
   - 选择合适的定位精度
   - 及时停止不需要的定位
   - 区分前后台定位策略

## 面试要点

1. 如何检测应用的耗电情况？
- 使用 Xcode Energy Log 工具
- 使用 Instruments 的 Energy Log 工具
- 监控 CPU 使用率
- 分析网络请求频率
- 检查定位服务使用情况
- 观察后台任务运行时长

2. 常见的耗电优化方案有哪些？
- 减少 CPU 密集型操作
- 合理使用定位服务
- 优化网络请求策略
- 管理后台任务
- 避免频繁的 I/O 操作
- 使用系统提供的省电 API

3. 如何优化后台任务？
- 使用 Background Task API
- 合并后台任务
- 设置合理的超时时间
- 及时结束后台任务
- 使用后台获取
- 实现任务优先级管理

4. 定位服务如何节省电量？
- 选择合适的定位精度
- 使用显著位置变化服务
- 区分前后台定位策略
- 及时停止定位更新
- 使用地理围栏服务
- 合理设置定位间隔

5. 网络请求对电量的影响？
- 频繁的请求增加耗电
- 长连接维护消耗电量
- 不当的重试策略
- 网络切换导致耗电
- 大量数据传输
- 后台网络请求

6. 如何监控应用的电量消耗？
- 使用 Energy Organizer
- 实现电量监控回调
- 记录重要操作的耗电
- 设置耗电阈值告警
- 分析耗电趋势
- 对比不同版本耗电

7. 有哪些耗电优化的工具和方法？
- Xcode Energy Debugger
- Instruments Energy Log
- Battery Usage Organizer
- MetricKit 框架
- Core Location 节能 API
- Background Task Framework

## 相关资源

- [Energy Efficiency Guide for iOS Apps](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/)
- [Energy Debugging with Xcode](https://developer.apple.com/videos/play/wwdc2017/238/)
- [Optimizing for Battery Life](https://developer.apple.com/videos/play/wwdc2019/417/) 