# UIKit 优化

## 基本概念

UIKit 优化主要包括界面渲染、布局计算和事件处理等方面的性能优化。主要包括：
- 视图层级优化
- 布局性能优化
- 渲染性能优化
- 事件处理优化

## 视图优化

### 1. 层级优化
```objc
// 减少视图层级
@interface OptimizedView : UIView

// 使用 CALayer 替代 UIView
@property (nonatomic, strong) CALayer *decorationLayer;
@property (nonatomic, strong) CATextLayer *textLayer;

- (void)setupLayers {
    // 使用 CALayer 而不是 UIView
    self.decorationLayer = [CALayer layer];
    self.decorationLayer.frame = CGRectMake(0, 0, 100, 100);
    self.decorationLayer.backgroundColor = [UIColor redColor].CGColor;
    [self.layer addSublayer:self.decorationLayer];
    
    // 使用 CATextLayer 而不是 UILabel
    self.textLayer = [CATextLayer layer];
    self.textLayer.string = @"Text";
    self.textLayer.fontSize = 16;
    [self.layer addSublayer:self.textLayer];
}

@end
```

### 2. 离屏渲染优化
```objc
// 避免离屏渲染
@implementation RenderOptimizedView

- (void)optimizeRendering {
    // 避免同时设置圆角和阴影
    self.layer.cornerRadius = 5;
    self.layer.masksToBounds = YES;
    
    // 使用 bezierPath 绘制阴影
    UIBezierPath *shadowPath = [UIBezierPath bezierPathWithRect:self.bounds];
    self.layer.shadowPath = shadowPath.CGPath;
    
    // 光栅化
    self.layer.shouldRasterize = YES;
    self.layer.rasterizationScale = [UIScreen mainScreen].scale;
}

@end
```

## 布局优化

### 1. 自动布局优化
```objc
// 布局约束优化
@interface LayoutOptimizedView : UIView

// 缓存布局结果
@property (nonatomic, assign) CGSize cachedIntrinsicContentSize;
@property (nonatomic, strong) NSLayoutConstraint *heightConstraint;

// 优化 intrinsicContentSize 计算
- (CGSize)intrinsicContentSize {
    if (CGSizeEqualToSize(self.cachedIntrinsicContentSize, CGSizeZero)) {
        // 计算内容大小
        self.cachedIntrinsicContentSize = [self calculateContentSize];
    }
    return self.cachedIntrinsicContentSize;
}

// 批量更新约束
- (void)updateConstraints {
    [NSLayoutConstraint deactivateConstraints:self.constraints];
    
    // 一次性添加所有约束
    NSMutableArray *constraints = [NSMutableArray array];
    [constraints addObject:[self.heightConstraint]];
    // 添加其他约束...
    
    [NSLayoutConstraint activateConstraints:constraints];
    
    [super updateConstraints];
}

@end
```

### 2. 手动布局优化
```objc
// 手动布局优化
@implementation ManualLayoutView

- (void)layoutSubviews {
    [super layoutSubviews];
    
    // 避免多次访问 frame
    CGRect bounds = self.bounds;
    CGFloat width = CGRectGetWidth(bounds);
    CGFloat height = CGRectGetHeight(bounds);
    
    // 一次性计算所有子视图位置
    CGFloat y = 0;
    for (UIView *subview in self.subviews) {
        CGFloat subviewHeight = [self heightForSubview:subview];
        subview.frame = CGRectMake(0, y, width, subviewHeight);
        y += subviewHeight;
    }
}

@end
```

## 事件处理

### 1. 响应链优化
```objc
// 事件处理优化
@implementation EventOptimizedView

// 优化点击区域
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    // 扩大点击区域
    CGRect hitArea = CGRectInset(self.bounds, -10, -10);
    return CGRectContainsPoint(hitArea, point);
}

// 优化事件传递
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    // 快速判断
    if (!self.userInteractionEnabled || self.hidden || self.alpha < 0.01) {
        return nil;
    }
    
    // 判断点是否在视图内
    if (![self pointInside:point withEvent:event]) {
        return nil;
    }
    
    // 从后往前遍历子视图
    for (UIView *subview in [self.subviews reverseObjectEnumerator]) {
        CGPoint convertedPoint = [self convertPoint:point toView:subview];
        UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
        if (hitTestView) {
            return hitTestView;
        }
    }
    
    return self;
}

@end
```

## 实践案例

### 1. 列表优化
```objc
// TableView 优化
@implementation OptimizedTableView

// 预计算行高
- (void)prepareRowHeight {
    self.estimatedRowHeight = 44;
    self.rowHeight = UITableViewAutomaticDimension;
}

// 异步绘制
+ (Class)layerClass {
    return [CATiledLayer class];
}

// 预加载
- (void)prefetchRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths {
    for (NSIndexPath *indexPath in indexPaths) {
        [self.dataSource prefetchDataAtIndex:indexPath.row];
    }
}

@end
```

### 2. 滚动优化
```objc
// 滚动优化
@implementation ScrollOptimizedView

// 减少离屏渲染
- (void)optimizeScrolling {
    // 设置合适的 tile size
    CATiledLayer *tiledLayer = (CATiledLayer *)self.layer;
    tiledLayer.tileSize = CGSizeMake(self.bounds.size.width, 44);
    
    // 预渲染
    self.layer.shouldRasterize = YES;
    self.layer.rasterizationScale = [UIScreen mainScreen].scale;
    
    // 异步绘制
    self.layer.drawsAsynchronously = YES;
}

@end
```

## 注意事项

1. **性能监控**
   - CPU 使用率
   - 内存占用
   - FPS 监测

2. **优化时机**
   - 按需加载
   - 延迟加载
   - 预加载

3. **调试工具**
   - Instruments
   - Core Animation
   - Time Profiler

## 面试要点

1. UIKit 性能优化的关键点？
- 减少视图层级复杂度
- 避免离屏渲染
- 优化布局计算
- 异步处理耗时操作
- 复用视图和资源

2. 如何优化视图层级？
- 使用 CALayer 替代 UIView
- 减少透明视图
- 合并相邻视图
- 使用 drawRect 绘制简单内容
- 避免视图嵌套过深

3. 自动布局vs手动布局？
- 自动布局更灵活但性能较差
- 手动布局性能好但维护成本高
- 复杂界面建议手动布局
- 简单界面可用自动布局
- 混合使用各取所长

4. 如何处理离屏渲染？
- 避免同时设置圆角和阴影
- 使用 CoreGraphics 绘制圆角
- 光栅化静态内容
- 使用 shadowPath 优化阴影
- 设置 opaque 属性

5. 列表性能优化方案？
- cell 复用机制
- 预排版和预计算
- 异步加载内容
- 按需加载
- 图片解码和缓存

6. 事件处理优化方式？
- 合理使用事件传递机制
- 避免频繁刷新
- 使用 GCD 处理耗时操作
- 控制响应链长度
- 减少事件响应对象

7. 如何监控 UI 性能？
- 使用 Instruments 工具
- FPS 监测
- CPU/内存使用率
- 卡顿检测
- 耗电量分析

## 相关资源

- [WWDC UIKit Sessions](https://developer.apple.com/videos/all-videos/?q=uikit)
- [Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)
- [Auto Layout Guide](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/index.html) 