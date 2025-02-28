# React Native

## 基本概念

React Native 是 Facebook 开发的跨平台框架，使用 JavaScript 开发原生应用。主要特点：
- 原生渲染：使用原生控件
- 热更新：动态更新 JS 代码
- 跨平台：一套代码多端运行
- Bridge 通信：JS 和原生通信机制

## 架构原理

### 1. Bridge 机制
```javascript
// JavaScript 代码
import { NativeModules } from 'react-native';

class NativeBridge {
  static async getData() {
    try {
      const result = await NativeModules.DataManager.getNativeData();
      return result;
    } catch (error) {
      console.error('Native call failed:', error);
      throw error;
    }
  }
}
```

```objc
// 原生模块
@interface DataManager : NSObject <RCTBridgeModule>
@end

@implementation DataManager

RCT_EXPORT_MODULE()

RCT_EXPORT_METHOD(getNativeData:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject) {
  // 执行原生操作
  NSString *data = @"Native Data";
  resolve(data);
}

@end
```

### 2. 渲染机制
```javascript
// React Native 组件
class CustomView extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.text}>Native Text</Text>
        <TouchableOpacity onPress={this.handlePress}>
          <Text>Click Me</Text>
        </TouchableOpacity>
      </View>
    );
  }

  handlePress = () => {
    // 触发原生事件
    NativeModules.EventEmitter.sendEvent('buttonClick');
  }
}
```

## 性能优化

### 1. 列表优化
```javascript
// FlatList 优化
class OptimizedList extends React.PureComponent {
  renderItem = ({ item }) => (
    <View style={styles.item}>
      <Text>{item.title}</Text>
    </View>
  );

  keyExtractor = item => item.id;

  getItemLayout = (data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  });

  render() {
    return (
      <FlatList
        data={this.props.data}
        renderItem={this.renderItem}
        keyExtractor={this.keyExtractor}
        getItemLayout={this.getItemLayout}
        removeClippedSubviews={true}
        initialNumToRender={10}
        maxToRenderPerBatch={10}
        windowSize={5}
      />
    );
  }
}
```

### 2. 内存优化
```javascript
// 图片优化
class ImageOptimization extends React.Component {
  render() {
    return (
      <Image
        source={{ uri: imageUrl }}
        style={styles.image}
        resizeMode="cover"
        onLoad={this.handleImageLoad}
        loadingIndicatorSource={require('./placeholder.png')}
      />
    );
  }

  handleImageLoad = () => {
    // 图片加载完成后的处理
  }
}
```

## 混合开发

### 1. 原生模块
```objc
// 自定义原生模块
@interface CustomModule : NSObject <RCTBridgeModule>
@end

@implementation CustomModule

RCT_EXPORT_MODULE()

// 导出方法
RCT_EXPORT_METHOD(handleCustomAction:(NSString *)action 
                  callback:(RCTResponseSenderBlock)callback) {
  // 处理原生操作
  dispatch_async(dispatch_get_main_queue(), ^{
    // 执行 UI 操作
    callback(@[[NSNull null], @"Success"]);
  });
}

// 导出常量
- (NSDictionary *)constantsToExport {
  return @{
    @"DEFAULT_TIMEOUT": @30,
    @"API_VERSION": @"1.0.0"
  };
}

@end
```

### 2. 原生 UI 组件
```objc
// 自定义原生视图
@interface CustomView : UIView
@end

@implementation CustomView
// 实现视图逻辑
@end

// 视图管理器
@interface CustomViewManager : RCTViewManager
@end

@implementation CustomViewManager

RCT_EXPORT_MODULE()

- (UIView *)view {
  return [[CustomView alloc] init];
}

// 导出属性
RCT_EXPORT_VIEW_PROPERTY(color, UIColor)
RCT_EXPORT_VIEW_PROPERTY(onPress, RCTBubblingEventBlock)

@end
```

## 注意事项

1. **性能考虑**
   - 减少 Bridge 通信
   - 优化列表渲染
   - 合理使用原生模块

2. **内存管理**
   - 及时清理监听器
   - 控制图片缓存
   - 避免内存泄漏

3. **调试技巧**
   - Chrome 调试器
   - React Native Debugger
   - 性能监控工具

## 面试要点

1. React Native 的工作原理？
- JS 引擎解析执行代码
- Bridge 实现跨语言通信
- 原生控件渲染界面
- Virtual DOM 更新机制
- 事件处理和回调

2. Bridge 的实现机制？
- 异步消息队列
- 序列化和反序列化
- 模块注册和导出
- 方法调用映射
- 批处理优化

3. 如何优化 RN 性能？
- 减少 Bridge 通信次数
- FlatList 替代 ScrollView
- 使用 PureComponent
- 合理设置缓存策略
- 避免重复渲染

4. 原生通信方式有哪些？
- RCTBridgeModule
- 回调函数(Callback)
- Promise
- 事件发送器
- 原生 UI 组件

5. 混合开发的最佳实践？
- 合理划分原生和 RN 页面
- 统一路由导航方案
- 封装通用基础组件
- 建立开发规范文档
- 版本管理和更新策略

6. RN 的优缺点分析？
优点:
- 跨平台开发效率高
- 动态更新能力
- 原生渲染性能好
- 组件复用性强
缺点:
- Bridge 通信开销
- 兼容性问题处理
- 框架升级成本高
- 原生功能受限

7. 热更新的实现原理？
- 远程加载 JS Bundle
- 代码签名验证
- 增量更新策略
- 版本回滚机制
- 异常处理和监控

## 相关资源

- [React Native Documentation](https://reactnative.dev/docs/getting-started)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [Native Modules](https://reactnative.dev/docs/native-modules-ios) 