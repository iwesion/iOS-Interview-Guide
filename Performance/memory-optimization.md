# 内存优化

## 基本概念

内存优化是指通过合理管理内存使用，避免内存泄漏、内存溢出等问题，提高应用的稳定性和性能。

## 内存问题

### 1. 内存泄漏
```objc
// 循环引用导致的内存泄漏
@interface ViewController ()
@property (nonatomic, copy) void (^block)(void);
@end

@implementation ViewController
- (void)viewDidLoad {
    self.block = ^{
        [self doSomething]; // self 被 block 强引用
    };
}
@end
```

### 2. 内存溢出
```objc
// 大量图片导致内存溢出
- (void)loadImages {
    NSMutableArray *images = [NSMutableArray array];
    for (int i = 0; i < 1000; i++) {
        UIImage *image = [UIImage imageNamed:@"large_image"];
        [images addObject:image];
    }
}
```

## 优化方案

### 1. 自动释放池
```objc
// 处理大量临时对象
for (int i = 0; i < count; i++) {
    @autoreleasepool {
        // 创建临时对象
        NSString *str = [NSString stringWithFormat:@"%d", i];
        [self processString:str];
    }
}
```

### 2. 图片优化
```objc
// 1. 图片解压缩
+ (UIImage *)decompressImage:(UIImage *)image {
    CGImageRef imageRef = image.CGImage;
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGContextRef context = CGBitmapContextCreate(NULL,
                                               CGImageGetWidth(imageRef),
                                               CGImageGetHeight(imageRef),
                                               8,
                                               0,
                                               colorSpace,
                                               kCGImageAlphaPremultipliedFirst);
    CGColorSpaceRelease(colorSpace);
    CGContextDrawImage(context, CGRectMake(0, 0, CGImageGetWidth(imageRef), CGImageGetHeight(imageRef)), imageRef);
    CGImageRef decompressedImageRef = CGBitmapContextCreateImage(context);
    CGContextRelease(context);
    UIImage *decompressedImage = [UIImage imageWithCGImage:decompressedImageRef];
    CGImageRelease(decompressedImageRef);
    return decompressedImage;
}

// 2. 图片缓存
- (void)cacheImage:(UIImage *)image forKey:(NSString *)key {
    NSCache *imageCache = [NSCache new];
    [imageCache setCountLimit:100];
    [imageCache setObject:image forKey:key];
}
```

### 3. 复用机制
```objc
// TableView 复用
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *CellIdentifier = @"Cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
    if (!cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
    }
    return cell;
}
```

## 实践案例

### 1. 大图片加载
```objc
@interface ImageLoader : NSObject

- (void)loadLargeImage:(NSString *)imageURL completion:(void(^)(UIImage *image))completion {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // 1. 下载图片
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:imageURL]];
        
        // 2. 压缩图片
        UIImage *image = [UIImage imageWithData:data];
        UIImage *thumbnailImage = [self compressImage:image toSize:CGSizeMake(100, 100)];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            if (completion) {
                completion(thumbnailImage);
            }
        });
    });
}

@end
```

### 2. 内存缓存管理
```objc
@interface MemoryCache : NSObject

@property (nonatomic, strong) NSCache *cache;

- (void)setupCache {
    self.cache = [[NSCache alloc] init];
    self.cache.countLimit = 100;
    self.cache.totalCostLimit = 10 * 1024 * 1024; // 10MB
    
    [[NSNotificationCenter defaultCenter] addObserver:self
                                           selector:@selector(clearCache)
                                               name:UIApplicationDidReceiveMemoryWarningNotification
                                             object:nil];
}

- (void)clearCache {
    [self.cache removeAllObjects];
}

@end
```

### 3. 页面内存优化
```objc
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // 1. 延迟加载
    [self performSelector:@selector(loadHeavyData) withObject:nil afterDelay:2.0];
    
    // 2. 分批加载
    [self loadDataWithBatch:0];
}

- (void)loadDataWithBatch:(NSInteger)batch {
    if (batch >= totalBatch) return;
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 加载数据
        dispatch_async(dispatch_get_main_queue(), ^{
            // 更新UI
            [self loadDataWithBatch:batch + 1];
        });
    });
}

- (void)dealloc {
    // 3. 清理资源
    [self.timer invalidate];
    self.timer = nil;
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

@end
```

## 监控方案

### 1. 内存使用监控
```objc
// 获取当前内存使用
- (double)getMemoryUsage {
    struct task_basic_info info;
    mach_msg_type_number_t size = sizeof(info);
    kern_return_t kerr = task_info(mach_task_self(), TASK_BASIC_INFO, (task_info_t)&info, &size);
    if (kerr == KERN_SUCCESS) {
        return info.resident_size / 1024.0 / 1024.0;
    }
    return 0;
}
```

### 2. 内存泄漏检测
```objc
// 使用 Instruments 的 Leaks 工具
// 使用 MLeaksFinder 等工具
```

## 注意事项

1. **资源管理**
   - 及时释放不需要的资源
   - 避免循环引用
   - 合理使用缓存

2. **大内存对象**
   - 避免大量图片同时加载
   - 使用延迟加载和分批加载
   - 合理设置缓存大小

3. **内存警告**
   - 监听内存警告通知
   - 实现低内存处理
   - 定期清理缓存

## 面试要点

1. 如何检测和解决内存泄漏？
- 使用 Instruments 的 Leaks/Allocations 工具
- 使用 MLeaksFinder 等三方工具
- 检查循环引用
- 使用 weak/assign 打破循环引用
- 正确使用自动释放池
- 检查 timer、通知、delegate 等资源是否释放

2. 大图片加载的优化方案？
- 异步解码图片
- 按需加载，延迟加载
- 图片压缩和裁剪
- 缓存解码后的图片
- 分批加载大量图片
- 监控内存使用阈值

3. 如何处理内存警告？
- 实现 didReceiveMemoryWarning 方法
- 清理图片、缓存等资源
- 释放可重新创建的对象
- 保存重要数据
- 降低图片质量
- 分级处理不同警告等级

4. TableView 的复用机制原理？
- 维护可重用队列
- 根据重用标识符复用 cell
- 离开屏幕的 cell 进入复用队列
- 需要显示时优先从队列获取
- 避免重复创建 cell
- 合理设置队列大小

5. NSCache 和 NSDictionary 的区别？
- NSCache 可自动清理内存
- NSCache 线程安全
- NSCache 可设置数量/成本限制
- NSCache 的 key 不会被 copy
- NSCache 适合缓存大对象
- NSCache 在内存警告时自动清理

6. 如何优化内存使用？
- 避免内存泄漏
- 及时释放不需要的资源
- 使用自动释放池
- 延迟加载、懒加载
- 合理使用缓存
- 图片压缩和按需加载

7. 有哪些内存监控方案？
- Instruments 工具监控
- 获取当前内存使用量
- 监听内存警告通知
- 自定义内存监控工具
- 设置内存使用阈值
- 分析内存增长趋势

## 相关资源

- [Memory Usage Performance Guidelines](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/ManagingMemory.html)
- [Advanced Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)
- [WWDC: iOS Memory Deep Dive](https://developer.apple.com/videos/play/wwdc2018/416/) 