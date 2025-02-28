# Core Animation

## 基本概念

Core Animation 是 iOS 的核心动画框架，提供高性能的图形渲染和动画效果。主要包括：
- 图层树：视图层级和呈现层级
- 隐式动画：属性动画
- 显式动画：自定义动画
- 性能优化：硬件加速

## 图层树

### 1. 层级结构
```objc
// 图层树结构
@interface LayerTreeExample : UIView

- (void)setupLayers {
    // 模型层
    CALayer *modelLayer = self.layer;
    
    // 呈现层
    CALayer *presentationLayer = self.layer.presentationLayer;
    
    // 添加子层
    CALayer *sublayer = [CALayer layer];
    sublayer.frame = CGRectMake(0, 0, 100, 100);
    sublayer.backgroundColor = [UIColor redColor].CGColor;
    [modelLayer addSublayer:sublayer];
}

@end
```

### 2. 图层属性
```objc
// 常用属性
@implementation LayerProperties

- (void)configureLayer {
    CALayer *layer = [CALayer layer];
    
    // 几何属性
    layer.frame = CGRectMake(0, 0, 100, 100);
    layer.position = CGPointMake(50, 50);
    layer.anchorPoint = CGPointMake(0.5, 0.5);
    
    // 视觉属性
    layer.backgroundColor = [UIColor redColor].CGColor;
    layer.opacity = 0.8;
    layer.cornerRadius = 10;
    layer.borderWidth = 1;
    layer.shadowOpacity = 0.5;
    
    // 内容属性
    layer.contents = (__bridge id)image.CGImage;
    layer.contentsGravity = kCAGravityResizeAspect;
    layer.masksToBounds = YES;
}

@end
```

## 动画实现

### 1. 隐式动画
```objc
// 属性动画
@implementation ImplicitAnimation

- (void)performAnimation {
    // 开启事务
    [CATransaction begin];
    [CATransaction setAnimationDuration:0.5];
    
    // 修改属性触发动画
    self.layer.position = CGPointMake(200, 200);
    self.layer.backgroundColor = [UIColor blueColor].CGColor;
    
    // 提交事务
    [CATransaction commit];
}

// 禁用隐式动画
- (void)disableImplicitAnimation {
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    self.layer.position = CGPointMake(200, 200);
    [CATransaction commit];
}

@end
```

### 2. 显式动画
```objc
// 基础动画
@implementation ExplicitAnimation

// 位置动画
- (void)addPositionAnimation {
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"position"];
    animation.fromValue = [NSValue valueWithCGPoint:CGPointMake(0, 0)];
    animation.toValue = [NSValue valueWithCGPoint:CGPointMake(200, 200)];
    animation.duration = 1.0;
    animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    [self.layer addAnimation:animation forKey:@"positionAnimation"];
}

// 关键帧动画
- (void)addKeyframeAnimation {
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path, NULL, 0, 0);
    CGPathAddCurveToPoint(path, NULL, 100, 0, 200, 200, 300, 200);
    
    animation.path = path;
    animation.duration = 2.0;
    animation.rotationMode = kCAAnimationRotateAuto;
    
    CGPathRelease(path);
    
    [self.layer addAnimation:animation forKey:@"pathAnimation"];
}

@end
```

## 性能优化

### 1. 绘制优化
```objc
// 异步绘制
@implementation AsyncDrawing

+ (Class)layerClass {
    return [CATiledLayer class];
}

- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
    // 在后台线程绘制
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        UIGraphicsPushContext(ctx);
        // 执行绘制
        UIGraphicsPopContext();
    });
}

@end
```

### 2. 动画优化
```objc
// 动画性能优化
@implementation AnimationOptimization

- (void)optimizeAnimation {
    // 使用 transform 代替 frame
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform"];
    animation.fromValue = [NSValue valueWithCATransform3D:CATransform3DIdentity];
    animation.toValue = [NSValue valueWithCATransform3D:CATransform3DMakeScale(2, 2, 1)];
    
    // 减少重绘
    self.layer.shouldRasterize = YES;
    self.layer.rasterizationScale = [UIScreen mainScreen].scale;
    
    // 避免离屏渲染
    self.layer.masksToBounds = NO;
    self.layer.cornerRadius = 0;
}

@end
```

## 实践案例

### 1. 自定义转场
```objc
// 转场动画
@interface CustomTransition : NSObject <CAAnimationDelegate>

- (void)animateTransition {
    CATransition *transition = [CATransition animation];
    transition.duration = 0.5;
    transition.type = kCATransitionPush;
    transition.subtype = kCATransitionFromRight;
    transition.delegate = self;
    
    [self.view.layer addAnimation:transition forKey:kCATransition];
}

@end
```

### 2. 粒子效果
```objc
// 粒子系统
@implementation ParticleEffect

- (void)setupEmitter {
    CAEmitterLayer *emitterLayer = [CAEmitterLayer layer];
    emitterLayer.emitterPosition = CGPointMake(self.bounds.size.width/2, 0);
    emitterLayer.emitterShape = kCAEmitterLayerLine;
    emitterLayer.emitterSize = CGSizeMake(self.bounds.size.width, 1);
    
    CAEmitterCell *cell = [[CAEmitterCell alloc] init];
    cell.contents = (__bridge id)[UIImage imageNamed:@"particle"].CGImage;
    cell.birthRate = 10;
    cell.lifetime = 5;
    cell.velocity = 100;
    cell.velocityRange = 50;
    cell.scale = 0.1;
    cell.scaleRange = 0.05;
    
    emitterLayer.emitterCells = @[cell];
    [self.layer addSublayer:emitterLayer];
}

@end
```

## 注意事项

1. **性能考虑**
   - 图层数量
   - 动画复杂度
   - 内存占用

2. **调试技巧**
   - 图层调试
   - 动画调试
   - 性能监控

3. **最佳实践**
   - 合理使用图层
   - 优化动画性能
   - 避免过度使用

## 面试要点

1. Core Animation 的基本原理？
- 基于图层的动画框架
- 运行在独立线程，不阻塞主线程
- 硬件加速，GPU 渲染
- 维护图层树结构
- 支持隐式和显式动画

2. 隐式动画和显式动画的区别？
- 隐式动画自动触发，修改属性即可
- 显式动画需要手动创建和添加
- 隐式动画只支持可动画属性
- 显式动画可以自定义动画过程
- 可以通过事务控制隐式动画

3. 如何优化动画性能？
- 减少图层数量和层级
- 避免离屏渲染
- 使用不透明图层
- 控制动画的复杂度
- 使用 shouldRasterize 缓存

4. 图层树的结构和作用？
- 模型层树：保存最终状态
- 呈现层树：显示当前状态
- 渲染层树：执行实际绘制
- 支持动画的插值计算
- 维护视图层级关系

5. 如何实现复杂动画效果？
- 组合基础动画
- 使用动画组
- 关键帧动画
- 自定义动画
- 粒子效果

6. 离屏渲染的原理和优化？
- GPU 在当前屏幕缓冲区以外开辟新缓冲区
- 先在离屏缓冲区渲染，再合成到当前屏幕
- 会导致性能损耗，应尽量避免
- 常见原因：圆角、阴影、遮罩
- 优化方案：光栅化、异步绘制

7. 常见的动画性能问题？
- 过多的图层和动画
- 频繁的离屏渲染
- 复杂的动画计算
- 内存占用过大
- 主线程阻塞

## 相关资源

- [Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)
- [WWDC Core Animation Sessions](https://developer.apple.com/videos/all-videos/?q=core+animation)
- [iOS Core Animation Advanced Techniques](https://www.amazon.com/iOS-Core-Animation-Advanced-Techniques-ebook/dp/B00EHJCORC) 