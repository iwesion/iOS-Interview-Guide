# 网络优化

## 基本概念

网络优化是提高应用网络性能的一系列措施，主要包括：
- 请求优化：减少请求次数、优化请求内容
- 连接优化：复用连接、预连接
- 数据优化：压缩、缓存策略
- 弱网优化：请求重试、降级策略

## 优化方案

### 1. 请求优化
```objc
// 请求合并
@interface BatchRequestManager : NSObject

@property (nonatomic, strong) NSMutableArray<Request *> *pendingRequests;
@property (nonatomic, assign) NSTimeInterval batchInterval;

- (void)addRequest:(Request *)request {
    [self.pendingRequests addObject:request];
    [self scheduleBatchRequestIfNeeded];
}

- (void)scheduleBatchRequestIfNeeded {
    if (self.pendingRequests.count >= 10) {
        [self executeBatchRequest];
    }
}

@end

// 请求优先级
typedef NS_ENUM(NSInteger, RequestPriority) {
    RequestPriorityLow,
    RequestPriorityDefault,
    RequestPriorityHigh
};

@interface Request : NSObject
@property (nonatomic, assign) RequestPriority priority;
@end
```

### 2. 连接优化
```objc
// 连接复用
@interface ConnectionPool : NSObject

@property (nonatomic, strong) NSMutableDictionary<NSString *, NSURLSession *> *sessions;

- (NSURLSession *)sessionForHost:(NSString *)host {
    NSURLSession *session = self.sessions[host];
    if (!session) {
        NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
        config.HTTPMaximumConnectionsPerHost = 6;
        config.timeoutIntervalForRequest = 15;
        session = [NSURLSession sessionWithConfiguration:config];
        self.sessions[host] = session;
    }
    return session;
}

@end

// 预连接
- (void)preconnect:(NSString *)host {
    NSURLSession *session = [self sessionForHost:host];
    [[[session dataTaskWithURL:[NSURL URLWithString:host]] resume];
}
```

### 3. 缓存策略
```objc
// 多级缓存
@interface CacheManager : NSObject

// 内存缓存
@property (nonatomic, strong) NSCache *memoryCache;
// 磁盘缓存
@property (nonatomic, strong) YYDiskCache *diskCache;

- (void)setObject:(id)object forKey:(NSString *)key {
    // 内存缓存
    [self.memoryCache setObject:object forKey:key];
    // 磁盘缓存
    [self.diskCache setObject:object forKey:key];
}

- (id)objectForKey:(NSString *)key {
    // 先查内存
    id object = [self.memoryCache objectForKey:key];
    if (object) return object;
    
    // 再查磁盘
    object = [self.diskCache objectForKey:key];
    if (object) {
        [self.memoryCache setObject:object forKey:key];
    }
    return object;
}

@end
```

## 实践案例

### 1. 图片加载优化
```objc
@interface ImageLoader : NSObject

// 渐进式加载
- (void)loadImageWithURL:(NSURL *)url progress:(void(^)(UIImage *))progress {
    NSURLSession *session = [NSURLSession sessionWithConfiguration:
                           [NSURLSessionConfiguration defaultSessionConfiguration]];
    
    [[session dataTaskWithURL:url completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        // 分块处理图片数据
        CGImageSourceRef source = CGImageSourceCreateIncremental(NULL);
        CGImageSourceUpdateData(source, (__bridge CFDataRef)data, false);
        
        // 逐步显示图片
        CGImageRef imageRef = CGImageSourceCreateImageAtIndex(source, 0, NULL);
        UIImage *image = [UIImage imageWithCGImage:imageRef];
        
        dispatch_async(dispatch_get_main_queue(), ^{
            progress(image);
        });
    }] resume];
}

@end
```

### 2. 弱网优化
```objc
@interface NetworkOptimizer : NSObject

// 请求重试
- (void)requestWithRetry:(NSURLRequest *)request 
              retryCount:(NSInteger)count 
             completion:(void(^)(id response, NSError *error))completion {
    [self executeRequest:request completion:^(id response, NSError *error) {
        if (error && count > 0) {
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC),
                         dispatch_get_main_queue(), ^{
                [self requestWithRetry:request 
                          retryCount:count - 1 
                         completion:completion];
            });
        } else {
            completion(response, error);
        }
    }];
}

// 降级策略
- (void)requestWithFallback:(NSArray<NSString *> *)APIs 
                completion:(void(^)(id response, NSError *error))completion {
    [self requestAPI:APIs[0] completion:^(id response, NSError *error) {
        if (error && APIs.count > 1) {
            NSArray *remainingAPIs = [APIs subarrayWithRange:NSMakeRange(1, APIs.count - 1)];
            [self requestWithFallback:remainingAPIs completion:completion];
        } else {
            completion(response, error);
        }
    }];
}

@end
```

## 注意事项

1. **请求控制**
   - 避免重复请求
   - 控制并发数
   - 请求优先级

2. **缓存设计**
   - 缓存策略
   - 缓存有效期
   - 缓存清理

3. **弱网处理**
   - 超时设置
   - 重试机制
   - 降级方案

## 面试要点

1. 如何优化网络请求？
- 请求合并减少请求次数
- 使用 HTTP/2 多路复用
- 合理设置缓存策略
- 请求优先级管理
- 预加载关键数据
- 压缩传输数据

2. 如何设计缓存策略？
- 内存缓存使用 NSCache
- 磁盘缓存使用 FileManager
- 设置合理的缓存时间
- 实现 LRU 淘汰算法
- 缓存分级管理
- 及时清理过期缓存

3. 如何处理弱网环境？
- 设置合理的超时时间
- 实现请求重试机制
- 准备降级接口方案
- 优化请求数据大小
- 本地缓存兜底
- 错误提示友好化

4. 如何优化图片加载？
- 异步下载和解码
- 根据设备压缩图片
- 使用合适的图片格式
- 预加载关键图片
- 缓存已加载图片
- 按需加载非关键图片

5. 如何实现请求优先级？
- 定义优先级等级
- 使用 NSOperationQueue 管理
- 高优先级请求优先执行
- 低优先级请求延迟执行
- 可取消低优先级请求
- 动态调整请求优先级

6. 如何处理并发请求？
- 使用 NSOperationQueue 控制
- 设置最大并发数
- 避免过度并发请求
- 合并相同的请求
- 取消无用的请求
- 监控请求队列状态

7. 如何设计重试机制？
- 设置最大重试次数
- 使用递增退避策略
- 只重试特定错误类型
- 避免无限重试
- 记录重试日志
- 重试时机选择合理

## 相关资源

- [URLSession Programming Guide](https://developer.apple.com/documentation/foundation/url_loading_system)
- [Network Link Conditioner](https://developer.apple.com/download/more/)
- [Mobile Networks Guide](https://developer.apple.com/library/archive/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/WhyNetworkingIsHard/WhyNetworkingIsHard.html) 