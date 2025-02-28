# 包体积优化

## 基本概念

包体积优化是指通过各种手段减小应用安装包(IPA)的大小，以提升下载转化率和用户体验。主要包括资源优化、代码优化等方面。

## 分析工具

### 1. LinkMap 分析
```objc
// 1. 开启 LinkMap
Build Settings -> Write Link Map File -> Yes

// 2. 分析工具
// 使用 LinkMap 分析工具查看各模块大小
```

### 2. AppStore Connect
```objc
// 查看 App Store 构建版本大小
// 分析各种架构的二进制大小
```

## 优化方案

### 1. 资源优化
```objc
// 1. 图片压缩
@implementation ImageCompressor

+ (void)compressImages {
    // 使用 TinyPNG API 压缩
    // 转换为 WebP 格式
    // 根据设备分辨率提供不同尺寸
}

// 2. 删除无用资源
// 使用 find . -name "*.png" | grep -i unused
// 使用工具扫描未使用的资源文件
```

### 2. 代码优化
```objc
// 1. 删除无用代码
#ifdef DEBUG
    // 仅在调试时使用的代码
#endif

// 2. 编译选项优化
// Build Settings
STRIP_STYLE = all
DEPLOYMENT_POSTPROCESSING = YES
DEAD_CODE_STRIPPING = YES
```

### 3. 架构优化
```objc
// 1. 支持的架构
// Build Settings -> Architectures
ARCHS = $(ARCHS_STANDARD)

// 2. bitcode
ENABLE_BITCODE = NO // 如果不需要可以关闭
```

## 实践案例

### 1. 资源按需加载
```objc
@implementation ResourceManager

- (void)loadResourcesForModule:(NSString *)module {
    // 1. 检查本地缓存
    if ([self hasLocalCache:module]) {
        return;
    }
    
    // 2. 从服务器下载
    [self downloadResources:module completion:^(BOOL success) {
        if (success) {
            [self saveToCache:module];
        }
    }];
}

@end
```

### 2. 动态库优化
```objc
// 1. 合并动态库
// 将功能相近的动态库合并

// 2. 转换为静态库
// 将不常更新的动态库转为静态库
```

### 3. 资源分包
```objc
@implementation PackageManager

- (void)setupPackages {
    // 1. 基础包
    // 包含启动必需资源
    
    // 2. 功能包
    // 按功能模块拆分，动态下载
    
    // 3. 差量更新
    // 仅更新变化的资源
}

@end
```

## 监控方案

### 1. 包大小监控
```objc
// 在 CI 流程中添加检查
if [[ $(stat -f%z "app.ipa") -gt 100000000 ]]; then
    echo "IPA size exceeds limit"
    exit 1
fi
```

### 2. 资源使用监控
```objc
@implementation ResourceMonitor

+ (void)checkUnusedResources {
    // 扫描项目中未使用的资源
    // 生成报告
}

+ (void)checkDuplicateResources {
    // 检查重复资源
    // MD5 对比
}

@end
```

## 注意事项

1. **资源管理**
   - 及时清理无用资源
   - 合理使用图片格式
   - 避免资源重复

2. **代码管理**
   - 删除无用代码
   - 合理使用编译选项
   - 注意库的依赖关系

3. **发布流程**
   - 建立包大小基线
   - 监控包大小变化
   - 制定优化策略

## 面试要点

1. 如何分析 App 的包大小构成？
- 使用 LinkMap 分析各模块大小
- 查看 AppStore Connect 构建版本大小
- 分析资源文件大小占比
- 检查动态库依赖情况
- 使用工具扫描未使用资源
- 分析不同架构的二进制大小

2. 有哪些减小包大小的方法？
- 压缩图片资源
- 删除无用代码和资源
- 优化动态库依赖
- 合理使用编译选项
- 使用 App Thinning
- 实现按需加载

3. 资源文件如何优化？
- 使用工具压缩图片
- 转换为 WebP 格式
- 提供不同分辨率版本
- 删除未使用资源
- 避免资源重复
- 使用矢量图形

4. 如何处理动态库依赖？
- 合并功能相似的库
- 删除未使用的库
- 控制库的版本
- 使用静态库替代
- 按需加载动态库
- 分析依赖关系

5. 包大小监控怎么做？
- CI 流程中添加检查
- 建立包大小基线
- 监控增量变化
- 设置告警阈值
- 定期生成分析报告
- 追踪历史趋势

6. 如何实现按需加载？
- 将非必要资源分离
- 使用动态下载
- 实现资源预加载
- 缓存已下载资源
- 监控下载进度
- 处理下载失败

7. 编译选项如何优化？
- 开启 Strip Dead Code
- 优化编译等级
- 配置 BitCode
- 设置压缩选项
- 管理调试信息
- 控制支持架构

## 相关资源

- [App Thinning in Xcode](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f)
- [Reducing Your App's Size](https://developer.apple.com/documentation/xcode/reducing-your-app-s-size)
- [Asset Catalogs](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/) 