# Flutter

## 基本概念

Flutter 是 Google 开发的开源 UI 框架，可以构建跨平台应用。主要特点：
- 单一代码库：使用 Dart 语言开发
- 自绘引擎：不依赖原生控件
- 高性能：接近原生的性能表现
- 热重载：快速开发调试

## 架构原理

### 1. 渲染机制
```dart
// Flutter 渲染流程
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: CustomPaint(
        painter: MyPainter(),
        child: Center(
          child: Text('Flutter Drawing'),
        ),
      ),
    );
  }
}

// 自定义绘制
class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2.0
      ..style = PaintingStyle.stroke;
      
    canvas.drawRect(
      Rect.fromLTWH(0, 0, size.width, size.height),
      paint
    );
  }
  
  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

### 2. 平台通道
```dart
// 原生通信
class PlatformChannel {
  static const platform = MethodChannel('com.example.app/channel');
  
  // 调用原生方法
  Future<void> getNativeData() async {
    try {
      final result = await platform.invokeMethod('getNativeData');
      print('Native result: $result');
    } catch (e) {
      print('Error: $e');
    }
  }
  
  // 接收原生调用
  void setupMethodCallHandler() {
    platform.setMethodCallHandler((call) async {
      switch (call.method) {
        case 'updateData':
          return handleDataUpdate(call.arguments);
        default:
          throw PlatformException(
            code: 'NotImplemented',
            message: 'Method not implemented'
          );
      }
    });
  }
}
```

## 性能优化

### 1. 渲染优化
```dart
// 避免重建
class OptimizedWidget extends StatelessWidget {
  const OptimizedWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return const RepaintBoundary(
      child: CustomScrollView(
        slivers: [
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => const ListItem(),
              childCount: 100,
            ),
          ),
        ],
      ),
    );
  }
}
```

### 2. 状态管理
```dart
// Provider 状态管理
class DataModel extends ChangeNotifier {
  String _data = '';
  
  String get data => _data;
  
  void updateData(String newData) {
    _data = newData;
    notifyListeners();
  }
}

// 使用 Provider
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => DataModel(),
      child: MaterialApp(
        home: MyHomePage(),
      ),
    );
  }
}
```

## 混合开发

### 1. 原生集成
```objc
// iOS 集成
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 创建 FlutterEngine
    self.flutterEngine = [[FlutterEngine alloc] initWithName:@"my_engine"];
    [self.flutterEngine run];
    
    // 注册平台通道
    FlutterMethodChannel *channel = [FlutterMethodChannel
        methodChannelWithName:@"com.example.app/channel"
              binaryMessenger:self.flutterEngine.binaryMessenger];
              
    [channel setMethodCallHandler:^(FlutterMethodCall* call, 
                                  FlutterResult result) {
        if ([@"getNativeData" isEqualToString:call.method]) {
            result(@"Native Data");
        } else {
            result(FlutterMethodNotImplemented);
        }
    }];
    
    return [super application:application 
        didFinishLaunchingWithOptions:launchOptions];
}

@end
```

### 2. 路由管理
```dart
// Flutter 路由
class AppRouter {
  static Route<dynamic> generateRoute(RouteSettings settings) {
    switch (settings.name) {
      case '/':
        return MaterialPageRoute(builder: (_) => HomePage());
      case '/detail':
        return MaterialPageRoute(builder: (_) => DetailPage());
      default:
        return MaterialPageRoute(
          builder: (_) => Scaffold(
            body: Center(
              child: Text('Route not found'),
            ),
          ),
        );
    }
  }
}
```

## 注意事项

1. **性能优化**
   - 合理使用 const 构造器
   - 避免不必要的重建
   - 使用 RepaintBoundary

2. **内存管理**
   - 及时释放资源
   - 避免内存泄漏
   - 图片缓存控制

3. **调试技巧**
   - 使用 DevTools
   - 性能分析
   - 内存监控

## 面试要点

1. Flutter 的渲染原理？
- 自绘引擎 Skia
- 不依赖原生控件
- 渲染流水线: UI线程 -> GPU线程 -> 光栅化
- Widget -> Element -> RenderObject 三层树结构
- 通过 VSync 信号同步刷新

2. Flutter 与原生通信机制？
- Platform Channel
- Method Channel: 方法调用
- Event Channel: 事件流
- Basic Message Channel: 基础消息
- 编解码器处理数据转换
- 异步通信模式

3. 如何优化 Flutter 性能？
- 合理使用 const 构造器
- 避免重建 Widget
- 使用 RepaintBoundary 隔离重绘
- 图片缓存管理
- 延迟加载和预加载
- 代码混淆和压缩

4. Flutter 状态管理方案？
- setState
- Provider
- Bloc
- Redux
- GetX
- Riverpod
- 选择合适的方案很重要

5. 混合开发注意事项？
- 路由管理
- 生命周期同步
- 数据传递
- 内存管理
- 版本兼容性
- 包大小控制

6. Flutter 优缺点分析？
优点:
- 跨平台
- 性能好
- 热重载
- 自绘引擎
- 丰富的组件库

缺点:
- 包体积大
- 学习成本
- 原生功能受限
- 三方库生态
- 调试相对困难

7. Flutter 实践经验？
- 架构设计
- 代码规范
- 组件封装
- 性能优化
- 测试策略
- CI/CD 流程
- 监控体系

## 相关资源

- [Flutter Documentation](https://flutter.dev/docs)
- [Flutter Performance](https://flutter.dev/docs/perf)
- [Flutter Platform Integration](https://flutter.dev/docs/development/platform-integration) 