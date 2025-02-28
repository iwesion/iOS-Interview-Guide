# Core Graphics

## 基本概念

Core Graphics 是 iOS 的核心绘图框架，提供 2D 渲染功能。主要包括：
- 绘图上下文：绘制环境
- 路径绘制：直线、曲线等
- 图形变换：旋转、缩放等
- 图片处理：裁剪、滤镜等

## 绘图基础

### 1. 上下文操作
```objc
// 绘图上下文
@implementation DrawingContext

- (void)drawRect:(CGRect)rect {
    // 获取当前上下文
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    // 保存状态
    CGContextSaveGState(context);
    
    // 设置绘图状态
    CGContextSetLineWidth(context, 2.0);
    CGContextSetStrokeColorWithColor(context, [UIColor redColor].CGColor);
    CGContextSetFillColorWithColor(context, [UIColor blueColor].CGColor);
    
    // 绘制操作...
    
    // 恢复状态
    CGContextRestoreGState(context);
}

@end
```

### 2. 路径绘制
```objc
// 基础图形
@implementation BasicShapes

- (void)drawShapes:(CGContextRef)context {
    // 矩形
    CGContextAddRect(context, CGRectMake(10, 10, 100, 100));
    
    // 圆形
    CGContextAddEllipseInRect(context, CGRectMake(120, 10, 100, 100));
    
    // 线条
    CGContextMoveToPoint(context, 10, 120);
    CGContextAddLineToPoint(context, 110, 220);
    
    // 贝塞尔曲线
    CGContextMoveToPoint(context, 120, 120);
    CGContextAddCurveToPoint(context, 140, 140, 160, 160, 220, 220);
    
    // 绘制路径
    CGContextStrokePath(context);
}

@end
```

## 图形处理

### 1. 变换操作
```objc
// 图形变换
@implementation GraphicsTransform

- (void)applyTransforms:(CGContextRef)context {
    // 平移
    CGContextTranslateCTM(context, 100, 100);
    
    // 旋转
    CGContextRotateCTM(context, M_PI_4);
    
    // 缩放
    CGContextScaleCTM(context, 1.5, 1.5);
    
    // 自定义变换
    CGAffineTransform transform = CGAffineTransformMakeScale(2.0, 2.0);
    transform = CGAffineTransformRotate(transform, M_PI_2);
    CGContextConcatCTM(context, transform);
}

@end
```

### 2. 渐变绘制
```objc
// 渐变效果
@implementation GradientDrawing

- (void)drawGradient:(CGContextRef)context {
    // 创建颜色空间
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    
    // 定义颜色
    CGFloat locations[] = {0.0, 1.0};
    CGFloat colors[] = {
        1.0, 0.0, 0.0, 1.0,  // 红色
        0.0, 0.0, 1.0, 1.0   // 蓝色
    };
    
    // 创建渐变
    CGGradientRef gradient = CGGradientCreateWithColorComponents(colorSpace, 
                                                               colors,
                                                               locations, 
                                                               2);
    
    // 绘制渐变
    CGPoint startPoint = CGPointMake(0, 0);
    CGPoint endPoint = CGPointMake(200, 200);
    CGContextDrawLinearGradient(context, 
                               gradient, 
                               startPoint, 
                               endPoint, 
                               kCGGradientDrawsBeforeStartLocation | 
                               kCGGradientDrawsAfterEndLocation);
    
    // 释放资源
    CGGradientRelease(gradient);
    CGColorSpaceRelease(colorSpace);
}

@end
```

## 图片处理

### 1. 图片绘制
```objc
// 图片操作
@implementation ImageProcessing

- (void)drawImage:(CGContextRef)context {
    UIImage *image = [UIImage imageNamed:@"example"];
    
    // 直接绘制
    [image drawInRect:CGRectMake(0, 0, 200, 200)];
    
    // 裁剪绘制
    CGContextClipToRect(context, CGRectMake(0, 0, 100, 100));
    CGContextDrawImage(context, CGRectMake(0, 0, 200, 200), image.CGImage);
    
    // 图片缩放
    UIGraphicsBeginImageContextWithOptions(CGSizeMake(100, 100), NO, 0);
    [image drawInRect:CGRectMake(0, 0, 100, 100)];
    UIImage *scaledImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
}

@end
```

### 2. 图片滤镜
```objc
// 滤镜效果
@implementation ImageFilters

- (UIImage *)applyFilter:(UIImage *)image {
    CIContext *context = [CIContext contextWithOptions:nil];
    CIImage *inputImage = [CIImage imageWithCGImage:image.CGImage];
    
    // 创建滤镜
    CIFilter *filter = [CIFilter filterWithName:@"CISepiaTone"];
    [filter setValue:inputImage forKey:kCIInputImageKey];
    [filter setValue:@0.8 forKey:kCIInputIntensityKey];
    
    // 获取输出图片
    CIImage *outputImage = filter.outputImage;
    CGImageRef cgImage = [context createCGImage:outputImage 
                                     fromRect:outputImage.extent];
    
    UIImage *result = [UIImage imageWithCGImage:cgImage];
    CGImageRelease(cgImage);
    
    return result;
}

@end
```

## 性能优化

### 1. 绘制优化
```objc
// 异步绘制
@implementation AsyncDrawing

- (void)asyncDraw {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, 0);
        // 执行绘制
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        
        dispatch_async(dispatch_get_main_queue(), ^{
            self.layer.contents = (__bridge id)image.CGImage;
        });
    });
}

@end
```

## 注意事项

1. **内存管理**
   - 及时释放资源
   - 控制图片大小
   - 重用上下文

2. **性能优化**
   - 异步绘制
   - 缓存结果
   - 避免过度绘制

3. **最佳实践**
   - 合理使用 API
   - 优化绘制次数
   - 注意内存泄漏

## 面试要点

1. Core Graphics 的基本原理？
- 基于 Quartz 2D 的绘图引擎
- 提供底层的 2D 渲染功能
- 使用 CPU 进行绘制
- 支持路径、图形、图片处理
- 基于状态机模式工作

2. 绘图上下文的作用？
- 保存绘图状态和配置
- 提供绘图操作的环境
- 管理绘图资源
- 控制绘制结果的输出
- 支持状态的保存和恢复

3. 如何优化绘制性能？
- 使用异步绘制
- 缓存绘制结果
- 控制重绘区域
- 避免透明图层
- 合理设置图片尺寸和压缩

4. 常见的绘图操作？
- 路径绘制（直线、曲线）
- 形状绘制（矩形、圆形）
- 图形变换（旋转、缩放）
- 渐变和阴影效果
- 文字和图片绘制

5. 图片处理的方法？
- 图片裁剪和缩放
- 图片格式转换
- 图片滤镜效果
- 图片合成
- 图片压缩和优化

6. 如何处理内存问题？
- 及时释放 CGContext
- 控制图片大小和数量
- 重用绘图上下文
- 避免频繁创建临时对象
- 使用自动释放池

7. 异步绘制的实现？
- 使用 GCD 异步队列
- 创建离屏上下文
- 在后台执行绘制
- 主线程更新界面
- 注意线程安全问题

## 相关资源

- [Quartz 2D Programming Guide](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Introduction/Introduction.html)
- [Core Graphics Framework](https://developer.apple.com/documentation/coregraphics)
- [WWDC Graphics Sessions](https://developer.apple.com/videos/all-videos/?q=graphics) 