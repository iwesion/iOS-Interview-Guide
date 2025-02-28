# 卡顿优化

## 基本概念

卡顿是指应用在运行过程中出现的界面不流畅、响应延迟等现象。主要原因是主线程被阻塞，导致无法及时响应用户操作和界面刷新。

## 卡顿原因

### 1. 主线程阻塞
```objc
// 在主线程执行耗时操作
- (void)viewDidLoad {
    [super viewDidLoad];
    [self heavyOperation]; // 耗时操作阻塞主线程
}
```

### 2. 过度绘制
```objc
// 多层视图叠加
- (void)setupViews {
    UIView *view1 = [[UIView alloc] init];
    view1.backgroundColor = [UIColor redColor];
    
    UIView *view2 = [[UIView alloc] init];
    view2.backgroundColor = [UIColor blueColor];
    
    [view1 addSubview:view2]; // 视图叠加导致过度绘制
}
```

## 优化方案

### 1. 异步处理
```objc
// 将耗时操作放到后台线程
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 耗时操作
    NSArray *results = [self heavyComputation];
    
    dispatch_async(dispatch_get_main_queue(), ^{
        // UI 更新
        [self updateUIWithResults:results];
    });
});
```

### 2. 视图优化
```objc
// 1. 预排版
+ (void)initialize {
    [super initialize];
    [WKWebView _prewarming];
}

// 2. 异步绘制
- (void)drawRect:(CGRect)rect {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 在后台线程进行绘制计算
        UIGraphicsBeginImageContextWithOptions(rect.size, NO, 0);
        // 绘制代码
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        
        dispatch_async(dispatch_get_main_queue(), ^{
            // 主线程更新
            self.layer.contents = (__bridge id)image.CGImage;
        });
    });
}
```

### 3. 数据优化
```objc
// 分页加载
- (void)loadDataWithPage:(NSInteger)page {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 加载数据
        NSArray *data = [self fetchDataWithPage:page];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            // 更新 UI
            [self.tableView reloadData];
        });
    });
}
```

## 监控方案

### 1. FPS 监控
```objc
@interface FPSMonitor : NSObject

@property (nonatomic, strong) CADisplayLink *displayLink;
@property (nonatomic, assign) NSTimeInterval lastTime;
@property (nonatomic, assign) NSInteger count;

- (void)start {
    self.displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(tick:)];
    [self.displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
}

- (void)tick:(CADisplayLink *)link {
    if (self.lastTime == 0) {
        self.lastTime = link.timestamp;
        return;
    }
    
    self.count++;
    NSTimeInterval delta = link.timestamp - self.lastTime;
    if (delta < 1) return;
    
    float fps = self.count / delta;
    self.count = 0;
    self.lastTime = link.timestamp;
    NSLog(@"FPS: %@", @(fps));
}

@end
```

### 2. 主线程监控
```objc
@interface MainThreadMonitor : NSObject

+ (void)startMonitoring {
    static dispatch_source_t timer;
    static dispatch_semaphore_t semaphore;
    
    if (timer) return;
    
    semaphore = dispatch_semaphore_create(0);
    timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(0, 0));
    
    dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC, 0);
    dispatch_source_set_event_handler(timer, ^{
        dispatch_async(dispatch_get_main_queue(), ^{
            dispatch_semaphore_signal(semaphore);
        });
        
        dispatch_time_t timeout = dispatch_time(DISPATCH_TIME_NOW, 100 * NSEC_PER_MSEC);
        if (dispatch_semaphore_wait(semaphore, timeout) != 0) {
            // 主线程卡顿，获取堆栈信息
            NSLog(@"Main thread blocked!");
        }
    });
    
    dispatch_resume(timer);
}

@end
```

## 实践案例

### 1. TableView 优化
```objc
@implementation TableViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 1. 预估高度
    self.tableView.estimatedRowHeight = 100;
    
    // 2. 异步绘制
    self.tableView.layer.drawsAsynchronously = YES;
    
    // 3. 预加载
    [self prefetchData];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    // 4. 缓存高度
    static NSMutableDictionary *heightCache;
    NSString *key = [NSString stringWithFormat:@"%ld-%ld", indexPath.section, indexPath.row];
    NSNumber *height = heightCache[key];
    if (!height) {
        height = @([self calculateHeightForRow:indexPath]);
        heightCache[key] = height;
    }
    return cell;
}

@end
```

### 2. 图片加载优化
```objc
@implementation ImageOptimizer

- (void)loadImage:(NSString *)url completion:(void(^)(UIImage *))completion {
    // 1. 检查缓存
    UIImage *cachedImage = [self.imageCache objectForKey:url];
    if (cachedImage) {
        completion(cachedImage);
        return;
    }
    
    // 2. 异步下载和处理
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:url]];
        UIImage *image = [UIImage imageWithData:data];
        
        // 3. 图片解码
        UIGraphicsBeginImageContextWithOptions(image.size, YES, 0);
        [image drawAtPoint:CGPointZero];
        image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        
        // 4. 缓存
        [self.imageCache setObject:image forKey:url];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            completion(image);
        });
    });
}

@end
```

## 注意事项

1. **主线程管理**
   - 避免主线程大量计算
   - 合理使用异步操作
   - 注意线程间通信

2. **资源管理**
   - 及时释放不需要的资源
   - 避免大量图片同时加载
   - 使用合适的缓存策略

3. **性能监控**
   - 监控 FPS
   - 监控主线程卡顿
   - 分析卡顿原因

## 面试要点

1. 如何检测和定位卡顿问题？
- 使用 Instruments 的 Time Profiler 工具分析 CPU 使用情况
- 监控主线程 RunLoop 的执行时间
- 使用 CADisplayLink 监控帧率
- 设置卡顿检测阈值进行报警

2. 列表滚动的优化方案有哪些？
- 异步绘制内容
- 预排版并缓存 cell 高度
- 减少透明视图
- 避免离屏渲染
- 按需加载，延迟加载
- 复用 cell 和视图

3. 如何优化图片加载？
- 异步解码图片
- 缓存解码后的图片
- 根据显示尺寸裁剪图片
- 优化图片存储格式
- 按需加载，延迟加载大图

4. 主线程卡顿的常见原因？
- 大量 I/O 操作
- 复杂 UI 绘制
- 大量计算任务
- 主线程网络请求
- 过度布局计算
- 频繁 AutoLayout

5. 如何实现 FPS 监控？
- 使用 CADisplayLink 计算帧率
- 监控 RunLoop 状态
- 记录掉帧情况
- 设置报警阈值

6. 如何处理大量数据的加载和显示？
- 分页加载
- 异步处理数据
- 预加载机制
- 数据缓存
- 增量更新
- 按需加载

7. 有哪些卡顿优化的工具和方法？
- Instruments (Time Profiler, Core Animation)
- Xcode Debug Navigator
- 自定义 FPS 监控工具
- 线上性能监控系统
- 静态分析工具
- 内存泄漏检测工具

## 相关资源

- [WWDC: Smooth Scrolling in TableViews and CollectionViews](https://developer.apple.com/videos/play/wwdc2018/219/)
- [Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)
- [Instruments Help: Time Profiler](https://help.apple.com/instruments/mac/current/#/dev44b2b437) 