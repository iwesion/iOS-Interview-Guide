# 网络协议

## 基本概念

网络协议是计算机网络中进行数据交换而建立的规则、标准或约定的集合。主要包括：
- TCP/IP：网络通信的基础协议
- HTTP/HTTPS：应用层协议
- WebSocket：全双工通信协议
- UDP：用户数据报协议

## 常用协议

### 1. HTTP/HTTPS
```objc
// HTTP 请求
NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];
request.URL = [NSURL URLWithString:@"https://api.example.com/users"];
request.HTTPMethod = @"POST";
request.allHTTPHeaderFields = @{
    @"Content-Type": @"application/json",
    @"Authorization": @"Bearer token"
};

// HTTPS 证书校验
@interface URLSessionDelegate : NSObject <NSURLSessionDelegate>
- (void)URLSession:(NSURLSession *)session
didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
 completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *))handler;
@end
```

### 2. TCP/IP
```objc
// Socket 编程
CFSocketRef socket = CFSocketCreate(kCFAllocatorDefault,
                                  PF_INET,
                                  SOCK_STREAM,
                                  IPPROTO_TCP,
                                  kCFSocketConnectCallBack,
                                  TCPClientCallBack,
                                  NULL);

// TCP 连接
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(8080);
addr.sin_addr.s_addr = inet_addr("192.168.1.1");
```

### 3. WebSocket
```objc
// WebSocket 连接
@interface WebSocketClient : NSObject <NSURLSessionWebSocketDelegate>
@property (nonatomic, strong) NSURLSessionWebSocketTask *webSocketTask;

- (void)connect {
    NSURL *url = [NSURL URLWithString:@"ws://example.com/socket"];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:
                           [NSURLSessionConfiguration defaultSessionConfiguration]
                                                        delegate:self
                                                   delegateQueue:nil];
    self.webSocketTask = [session webSocketTaskWithURL:url];
    [self.webSocketTask resume];
}

- (void)sendMessage:(NSString *)message {
    NSURLSessionWebSocketMessage *webSocketMessage = 
        [[NSURLSessionWebSocketMessage alloc] initWithString:message];
    [self.webSocketTask sendMessage:webSocketMessage 
                  completionHandler:^(NSError *error) {
        // 处理发送结果
    }];
}
@end
```

## 协议特点

### 1. HTTP vs HTTPS
```objc
// HTTP
- 明文传输
- 无需证书
- 性能较好
- 不安全

// HTTPS
- 加密传输
- 需要证书
- 性能损耗
- 更安全
```

### 2. TCP vs UDP
```objc
// TCP
- 面向连接
- 可靠传输
- 有序传输
- 流量控制

// UDP
- 无连接
- 不可靠
- 高效传输
- 适合实时数据
```

## 实践案例

### 1. 网络请求封装
```objc
@interface NetworkManager : NSObject

// GET 请求
- (void)GET:(NSString *)URLString
 parameters:(NSDictionary *)parameters
completion:(void(^)(id response, NSError *error))completion;

// POST 请求
- (void)POST:(NSString *)URLString
  parameters:(NSDictionary *)parameters
 completion:(void(^)(id response, NSError *error))completion;

// 文件上传
- (void)uploadFile:(NSData *)fileData
              name:(NSString *)name
         mimeType:(NSString *)mimeType
       completion:(void(^)(id response, NSError *error))completion;

@end
```

### 2. 长连接实现
```objc
@interface SocketManager : NSObject

// 心跳检测
- (void)startHeartbeat {
    self.heartbeatTimer = [NSTimer scheduledTimerWithTimeInterval:30
                                                         repeats:YES
                                                           block:^(NSTimer *timer) {
        [self sendHeartbeat];
    }];
}

// 自动重连
- (void)autoReconnect {
    if (self.reconnectCount < MAX_RECONNECT_COUNT) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 
                                  RECONNECT_INTERVAL * NSEC_PER_SEC),
                      dispatch_get_main_queue(), ^{
            [self connect];
            self.reconnectCount++;
        });
    }
}

@end
```

## 注意事项

1. **安全性**
   - 使用 HTTPS
   - 证书校验
   - 数据加密

2. **性能优化**
   - 连接复用
   - 请求合并
   - 压缩传输

3. **稳定性**
   - 错误重试
   - 超时处理
   - 弱网优化

## 面试要点

1. HTTP 和 HTTPS 的区别？
- HTTP 是明文传输，HTTPS 是加密传输
- HTTPS 需要 CA 证书
- HTTPS 默认端口 443，HTTP 默认端口 80
- HTTPS 需要 SSL/TLS 握手
- HTTPS 更安全但性能略低

2. TCP 的三次握手和四次挥手？
- 三次握手:
  1. 客户端发送 SYN
  2. 服务端回复 SYN+ACK
  3. 客户端发送 ACK
- 四次挥手:
  1. 客户端发送 FIN
  2. 服务端回复 ACK
  3. 服务端发送 FIN
  4. 客户端回复 ACK

3. WebSocket 的优势和应用场景？
- 全双工通信，服务端可主动推送
- 只需要一次 HTTP 握手
- 数据格式轻量，性能开销小
- 适用于:
  - 即时通讯
  - 实时数据展示
  - 多人游戏
  - 协同编辑

4. 如何处理网络安全问题？
- 使用 HTTPS 加密传输
- 实现证书校验(SSL Pinning)
- 敏感数据加密
- 防重放攻击
- 检测代理抓包
- 混淆网络数据

5. 如何优化网络性能？
- 使用 HTTP/2
- 启用 GZip 压缩
- 合理设置缓存
- 请求合并
- 连接复用
- DNS 优化
- 图片优化

6. 如何处理弱网环境？
- 请求超时设置
- 断线重连机制
- 请求优先级
- 本地缓存
- 降级策略
- 分片上传
- 错误重试

7. Socket 编程的要点？
- 正确配置 Socket 参数
- 实现心跳机制
- 处理断线重连
- 数据粘包处理
- 缓冲区管理
- 超时处理
- 错误处理

## 相关资源

- [HTTP/2](https://developers.google.com/web/fundamentals/performance/http2)
- [TCP/IP Guide](http://www.tcpipguide.com/)
- [WebSocket Protocol](https://tools.ietf.org/html/rfc6455) 