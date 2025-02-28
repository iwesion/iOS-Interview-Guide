# 网络安全

## 基本概念

网络安全是保护应用程序免受网络攻击和数据泄露的一系列措施。主要包括：
- 传输安全：HTTPS、SSL Pinning
- 数据安全：加密、签名
- 防护措施：防抓包、防重放
- 身份认证：Token、证书

## 安全方案

### 1. HTTPS 配置
```objc
// SSL Pinning
@interface SecurityPolicy : NSObject

@property (nonatomic, strong) NSSet<NSData *> *pinnedCertificates;

- (BOOL)evaluateServerTrust:(SecTrustRef)serverTrust 
                  forDomain:(NSString *)domain {
    // 获取服务器证书
    SecCertificateRef certificate = SecTrustGetCertificateAtIndex(serverTrust, 0);
    NSData *serverCertificateData = (__bridge_transfer NSData *)SecCertificateCopyData(certificate);
    
    // 验证证书
    return [self.pinnedCertificates containsObject:serverCertificateData];
}

// 加载证书
- (void)loadCertificates {
    NSString *certPath = [[NSBundle mainBundle] pathForResource:@"cert" ofType:@"cer"];
    NSData *certData = [NSData dataWithContentsOfFile:certPath];
    self.pinnedCertificates = [NSSet setWithObject:certData];
}

@end
```

### 2. 数据加密
```objc
// 加密工具
@interface CryptoUtil : NSObject

// AES 加密
+ (NSData *)AES256EncryptData:(NSData *)data withKey:(NSString *)key {
    char keyPtr[kCCKeySizeAES256 + 1];
    bzero(keyPtr, sizeof(keyPtr));
    [key getCString:keyPtr maxLength:sizeof(keyPtr) encoding:NSUTF8StringEncoding];
    
    NSUInteger dataLength = [data length];
    size_t bufferSize = dataLength + kCCBlockSizeAES128;
    void *buffer = malloc(bufferSize);
    
    size_t numBytesEncrypted = 0;
    CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt,
                                        kCCAlgorithmAES128,
                                        kCCOptionPKCS7Padding,
                                        keyPtr,
                                        kCCKeySizeAES256,
                                        NULL,
                                        [data bytes],
                                        dataLength,
                                        buffer,
                                        bufferSize,
                                        &numBytesEncrypted);
    
    if (cryptStatus == kCCSuccess) {
        return [NSData dataWithBytesNoCopy:buffer length:numBytesEncrypted];
    }
    free(buffer);
    return nil;
}

// RSA 签名
+ (NSString *)RSASignData:(NSData *)data withPrivateKey:(SecKeyRef)privateKey {
    size_t signedHashBytesSize = SecKeyGetBlockSize(privateKey);
    uint8_t *signedHashBytes = malloc(signedHashBytesSize);
    
    SecKeyRawSign(privateKey,
                  kSecPaddingPKCS1SHA256,
                  data.bytes,
                  data.length,
                  signedHashBytes,
                  &signedHashBytesSize);
    
    NSData *signedHash = [NSData dataWithBytes:signedHashBytes length:signedHashBytesSize];
    free(signedHashBytes);
    return [signedHash base64EncodedStringWithOptions:0];
}

@end
```

## 防护措施

### 1. 防抓包
```objc
@interface SecurityManager : NSObject

// 检测代理
- (BOOL)detectProxy {
    CFDictionaryRef proxySettings = CFNetworkCopySystemProxySettings();
    NSDictionary *settings = (__bridge_transfer NSDictionary *)proxySettings;
    return [settings[@"HTTPEnable"] boolValue];
}

// 检测越狱
- (BOOL)detectJailbreak {
    NSArray *paths = @[
        @"/Applications/Cydia.app",
        @"/Library/MobileSubstrate/MobileSubstrate.dylib",
        @"/bin/bash",
        @"/usr/sbin/sshd"
    ];
    
    for (NSString *path in paths) {
        if ([[NSFileManager defaultManager] fileExistsAtPath:path]) {
            return YES;
        }
    }
    return NO;
}

// 防重放攻击
- (BOOL)validateRequest:(NSURLRequest *)request {
    NSString *timestamp = request.allHTTPHeaderFields[@"Timestamp"];
    NSString *nonce = request.allHTTPHeaderFields[@"Nonce"];
    NSString *signature = request.allHTTPHeaderFields[@"Signature"];
    
    // 验证时间戳
    NSTimeInterval now = [[NSDate date] timeIntervalSince1970];
    if (now - [timestamp doubleValue] > 300) { // 5分钟过期
        return NO;
    }
    
    // 验证 nonce
    if ([self.usedNonces containsObject:nonce]) {
        return NO;
    }
    
    // 验证签名
    return [self verifySignature:signature forRequest:request];
}

@end
```

### 2. 数据保护
```objc
@interface DataProtector : NSObject

// 敏感数据加密存储
- (void)secureStoreData:(NSString *)data forKey:(NSString *)key {
    NSData *encryptedData = [CryptoUtil AES256EncryptData:[data dataUsingEncoding:NSUTF8StringEncoding]
                                                 withKey:self.encryptionKey];
    
    NSDictionary *query = @{
        (__bridge id)kSecClass: (__bridge id)kSecClassGenericPassword,
        (__bridge id)kSecAttrAccount: key,
        (__bridge id)kSecValueData: encryptedData
    };
    
    SecItemAdd((__bridge CFDictionaryRef)query, NULL);
}

// 防止截屏
- (void)preventScreenCapture {
    [[NSNotificationCenter defaultCenter] addObserver:self
                                           selector:@selector(handleScreenshot)
                                               name:UIApplicationUserDidTakeScreenshotNotification
                                             object:nil];
}

@end
```

## 注意事项

1. **证书安全**
   - 证书固定
   - 证书验证
   - 证书更新

2. **数据安全**
   - 传输加密
   - 存储加密
   - 敏感信息保护

3. **防护措施**
   - 代码混淆
   - 反调试
   - 完整性校验

## 面试要点

1. HTTPS 的工作原理？
- 基于 HTTP 协议,通过 SSL/TLS 加密
- 工作流程:
  1. 客户端发起 HTTPS 请求
  2. 服务器返回证书
  3. 客户端验证证书
  4. 协商对称加密密钥
  5. 使用对称密钥加密通信

2. 如何实现 SSL Pinning？
- 证书固定方式:
  - 保存证书到 App Bundle
  - 代码验证证书有效性
  - 对比证书是否匹配
- 公钥固定方式:
  - 保存公钥哈希
  - 验证服务器证书公钥

3. 常见的加密算法有哪些？
- 对称加密:
  - AES
  - DES
  - 3DES
- 非对称加密:
  - RSA
  - ECC
- 哈希算法:
  - MD5
  - SHA1/SHA256

4. 如何防止抓包攻击？
- 实现证书固定(SSL Pinning)
- 检测代理设置
- 加密传输数据
- 混淆网络数据
- 反调试保护

5. 如何保护敏感数据？
- 使用 Keychain 存储
- 加密存储敏感信息
- 内存数据及时清除
- 防止截屏/录屏
- 禁止数据备份

6. 如何防止重放攻击？
- 时间戳机制
- 随机数(nonce)
- 请求签名
- 会话令牌
- 一次性令牌

7. 如何做安全加固？
- 代码混淆和加密
- 反调试和越狱检测
- 完整性校验
- 安全通信
- 数据安全存储
- 运行时保护

## 相关资源

- [OWASP Mobile Security](https://owasp.org/www-project-mobile-security/)
- [Apple Security Guide](https://developer.apple.com/documentation/security)
- [Common Crypto](https://developer.apple.com/documentation/security/common_crypto) 