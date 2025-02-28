# 抓包原理

## 基本概念

抓包是通过代理服务器拦截、分析和修改网络请求的过程。主要包括：
- HTTP 代理：拦截 HTTP/HTTPS 请求
- TCP 代理：拦截 TCP 连接
- SSL/TLS：HTTPS 证书和解密
- 数据分析：请求响应分析

## 工作原理

### 1. 代理设置
```objc
// 系统代理设置
NSDictionary *proxySettings = @{
    (NSString *)kCFProxyHostNameKey: @"localhost",
    (NSString *)kCFProxyPortNumberKey: @8888,
    (NSString *)kCFProxyTypeKey: (NSString *)kCFProxyTypeHTTP
};

// 应用代理配置
NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
config.connectionProxyDictionary = proxySettings;
```

### 2. HTTPS 解密
```objc
// 证书信任
@interface CustomURLSessionDelegate : NSObject <NSURLSessionDelegate>

- (void)URLSession:(NSURLSession *)session
didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
 completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler {
    if ([challenge.protectionSpace.authenticationMethod 
         isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        // 信任代理证书
        NSURLCredential *credential = [NSURLCredential credentialForTrust:
                                     challenge.protectionSpace.serverTrust];
        completionHandler(NSURLSessionAuthChallengeUseCredential, credential);
    }
}

@end
```

## 实现方式

### 1. 代理服务器
```objc
// 本地代理服务器
@interface ProxyServer : NSObject

@property (nonatomic, assign) uint16_t port;
@property (nonatomic, strong) GCDAsyncSocket *serverSocket;

- (void)startServer {
    self.serverSocket = [[GCDAsyncSocket alloc] initWithDelegate:self 
                                                  delegateQueue:dispatch_get_main_queue()];
    NSError *error = nil;
    if ([self.serverSocket acceptOnPort:self.port error:&error]) {
        NSLog(@"代理服务器启动成功，端口：%d", self.port);
    }
}

// 处理连接
- (void)socket:(GCDAsyncSocket *)sock didAcceptNewSocket:(GCDAsyncSocket *)newSocket {
    // 处理新的连接请求
    ProxyConnection *connection = [[ProxyConnection alloc] initWithSocket:newSocket];
    [connection start];
}

@end
```

### 2. 请求拦截
```objc
// 请求拦截器
@interface RequestInterceptor : NSObject

// 拦截请求
- (void)interceptRequest:(NSURLRequest *)request completion:(void(^)(NSURLRequest *))completion {
    NSMutableURLRequest *mutableRequest = [request mutableCopy];
    
    // 修改请求头
    [mutableRequest setValue:@"Modified User-Agent" forHTTPHeaderField:@"User-Agent"];
    
    // 修改请求体
    if ([request.HTTPMethod isEqualToString:@"POST"]) {
        NSMutableDictionary *params = [NSJSONSerialization 
                                     JSONObjectWithData:request.HTTPBody 
                                     options:0 error:nil].mutableCopy;
        params[@"intercepted"] = @YES;
        mutableRequest.HTTPBody = [NSJSONSerialization dataWithJSONObject:params 
                                                                options:0 
                                                                  error:nil];
    }
    
    completion(mutableRequest);
}

@end
```

## 实践案例

### 1. 网络调试
```objc
@interface NetworkDebugger : NSObject

// 记录请求
- (void)logRequest:(NSURLRequest *)request {
    NSLog(@"URL: %@", request.URL);
    NSLog(@"Method: %@", request.HTTPMethod);
    NSLog(@"Headers: %@", request.allHTTPHeaderFields);
    NSLog(@"Body: %@", [[NSString alloc] initWithData:request.HTTPBody 
                                           encoding:NSUTF8StringEncoding]);
}

// 记录响应
- (void)logResponse:(NSURLResponse *)response data:(NSData *)data {
    NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
    NSLog(@"Status: %ld", (long)httpResponse.statusCode);
    NSLog(@"Headers: %@", httpResponse.allHeaderFields);
    NSLog(@"Body: %@", [[NSString alloc] initWithData:data 
                                           encoding:NSUTF8StringEncoding]);
}

@end
```

### 2. 安全测试
```objc
@interface SecurityTester : NSObject

// 证书校验
- (BOOL)validateCertificate:(SecCertificateRef)certificate {
    SecTrustRef trust = NULL;
    SecTrustCreateWithCertificates(certificate, NULL, &trust);
    
    SecTrustResultType result;
    SecTrustEvaluate(trust, &result);
    
    return result == kSecTrustResultUnspecified || 
           result == kSecTrustResultProceed;
}

// 检测代理
- (BOOL)isProxyEnabled {
    CFDictionaryRef proxySettings = CFNetworkCopySystemProxySettings();
    NSDictionary *settings = (__bridge_transfer NSDictionary *)proxySettings;
    return [settings[@"HTTPEnable"] boolValue] || 
           [settings[@"HTTPSEnable"] boolValue];
}

@end
```

## 注意事项

1. **安全性**
   - 证书校验
   - 代理检测
   - 数据加密

2. **性能影响**
   - 网络延迟
   - 内存占用
   - CPU 消耗

3. **调试技巧**
   - 过滤请求
   - 断点调试
   - 数据分析

## 面试要点

1. 抓包的基本原理是什么？
   - 通过代理服务器拦截网络请求
   - 解析 HTTP/HTTPS 协议数据
   - 查看和分析请求/响应内容

2. 如何处理 HTTPS 抓包？
   - 安装 CA 证书到设备
   - 配置代理服务器
   - 中间人攻击原理
   - SSL Pinning 绕过

3. 如何防止抓包？
   - 证书校验(SSL Pinning)
   - 代理检测
   - 数据加密
   - 反调试技术

4. Charles 的工作原理？
   - 作为系统代理
   - SSL 代理功能
   - 请求重写与断点
   - Map Local/Remote 功能

5. 如何进行安全测试？
   - 证书验证测试
   - 代理检测
   - 加密算法测试
   - 敏感信息检查

6. 如何分析网络性能？
   - 请求耗时统计
   - 网络带宽分析
   - 请求优化建议
   - 缓存策略评估

7. 抓包工具的实现原理？
   - 代理服务器实现
   - 证书管理
   - 协议解析
   - 界面展示

## 相关资源

- [Charles Proxy](https://www.charlesproxy.com/)
- [Wireshark](https://www.wireshark.org/)
- [MITM Proxy](https://mitmproxy.org/) 