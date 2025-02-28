# SwiftUI

## 基本概念

SwiftUI 是 Apple 推出的声明式 UI 框架，用于构建 iOS、macOS 等平台的用户界面。主要特点：
- 声明式语法：简洁直观的 UI 代码
- 数据驱动：自动更新 UI
- 跨平台：支持所有 Apple 平台
- 实时预览：快速开发调试

## 核心组件

### 1. 视图构建
```swift
// 基本视图
struct ContentView: View {
    @State private var text = "Hello"
    
    var body: some View {
        VStack(spacing: 20) {
            Text(text)
                .font(.title)
                .foregroundColor(.blue)
            
            Button("Update") {
                text = "Updated"
            }
            .padding()
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(8)
        }
    }
}
```

### 2. 状态管理
```swift
// 状态和绑定
class UserData: ObservableObject {
    @Published var username = ""
    @Published var isLoggedIn = false
}

struct LoginView: View {
    @StateObject private var userData = UserData()
    @State private var password = ""
    
    var body: some View {
        Form {
            TextField("Username", text: $userData.username)
            SecureField("Password", text: $password)
            
            Button("Login") {
                self.login()
            }
        }
    }
    
    private func login() {
        // 登录逻辑
        userData.isLoggedIn = true
    }
}
```

## 布局系统

### 1. 自适应布局
```swift
// 响应式布局
struct AdaptiveView: View {
    @Environment(\.horizontalSizeClass) var sizeClass
    
    var body: some View {
        Group {
            if sizeClass == .compact {
                VStack {
                    content
                }
            } else {
                HStack {
                    content
                }
            }
        }
    }
    
    var content: some View {
        Group {
            Image(systemName: "star.fill")
            Text("Content")
        }
    }
}
```

### 2. 自定义布局
```swift
// 自定义布局
struct CustomLayout: Layout {
    func sizeThatFits(proposal: ProposedViewSize,
                     subviews: Subviews,
                     cache: inout Void) -> CGSize {
        // 计算布局大小
        return CGSize(width: 200, height: 200)
    }
    
    func placeSubviews(in bounds: CGRect,
                      proposal: ProposedViewSize,
                      subviews: Subviews,
                      cache: inout Void) {
        // 放置子视图
        for (index, subview) in subviews.enumerated() {
            let point = CGPoint(x: bounds.minX + CGFloat(index) * 50,
                              y: bounds.minY)
            subview.place(at: point, proposal: .unspecified)
        }
    }
}
```

## 性能优化

### 1. 视图优化
```swift
// 视图性能优化
struct OptimizedView: View {
    let items: [Item]
    
    var body: some View {
        List(items) { item in
            ItemRow(item: item)
                .listRowInsets(EdgeInsets())
                .drawingGroup() // 启用离屏渲染
        }
    }
}

struct ItemRow: View {
    let item: Item
    
    var body: some View {
        HStack {
            AsyncImage(url: item.imageURL) { image in
                image.resizable()
                     .aspectRatio(contentMode: .fill)
            } placeholder: {
                ProgressView()
            }
            .frame(width: 50, height: 50)
            
            Text(item.title)
        }
    }
}
```

### 2. 数据流优化
```swift
// 状态管理优化
final class ViewModel: ObservableObject {
    @Published private(set) var items: [Item] = []
    private var cancellables = Set<AnyCancellable>()
    
    func fetchItems() {
        // 使用 Combine 处理数据流
        URLSession.shared.dataTaskPublisher(for: URL(string: "api/items")!)
            .map { $0.data }
            .decode(type: [Item].self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .sink { completion in
                // 处理完成
            } receiveValue: { [weak self] items in
                self?.items = items
            }
            .store(in: &cancellables)
    }
}
```

## 与 UIKit 集成

### 1. UIKit 桥接
```swift
// UIKit 视图封装
struct UIKitView: UIViewRepresentable {
    func makeUIView(context: Context) -> UIView {
        let view = UIView()
        view.backgroundColor = .red
        return view
    }
    
    func updateUIView(_ uiView: UIView, context: Context) {
        // 更新视图
    }
}

// UIKit 控制器封装
struct UIKitViewController: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        let viewController = UIViewController()
        // 配置视图控制器
        return viewController
    }
    
    func updateUIViewController(_ uiViewController: UIViewController,
                              context: Context) {
        // 更新视图控制器
    }
}
```

## 注意事项

1. **性能考虑**
   - 避免频繁状态更新
   - 合理使用 drawingGroup
   - 优化数据流

2. **内存管理**
   - 注意循环引用
   - 及时取消订阅
   - 合理使用 weak self

3. **调试技巧**
   - 使用 Preview
   - Instruments 分析
   - View Hierarchy 调试

## 面试要点

1. SwiftUI 的工作原理？
- 声明式 UI 框架
- 数据驱动视图更新
- 基于属性包装器实现状态管理
- 视图是值类型
- 自动差异化更新

2. 状态管理方式有哪些？
- @State: 视图内部状态
- @Binding: 状态传递和共享
- @ObservedObject: 外部可观察对象
- @StateObject: 视图拥有的可观察对象
- @EnvironmentObject: 环境注入
- @Environment: 环境值读取

3. 如何优化 SwiftUI 性能？
- 减少视图层级
- 使用 drawingGroup() 开启 Metal 渲染
- 避免频繁状态更新
- 合理使用 id() 标识
- LazyVStack/LazyHStack 延迟加载
- 优化计算属性和方法

4. 与 UIKit 如何混合开发？
- UIViewRepresentable 封装 UIView
- UIViewControllerRepresentable 封装 UIViewController
- 使用 Coordinator 处理代理
- 通过 hosting controller 在 UIKit 中使用 SwiftUI
- 状态同步和数据传递

5. SwiftUI 的生命周期？
- onAppear/onDisappear
- task/onReceive 异步任务
- onChange 状态变化
- onReceive 通知监听
- 视图更新周期

6. 数据流的处理方式？
- 单向数据流
- 发布者-订阅者模式
- Combine 框架集成
- 环境传值
- 状态提升

7. SwiftUI 的优缺点？
优点:
- 声明式语法简洁
- 自动管理状态
- 实时预览
- 跨平台支持
- 响应式设计
缺点:
- 学习曲线陡峭
- 部分 UIKit 功能缺失
- iOS 13+ 限制
- 复杂交互实现困难
- 调试体验欠佳

## 相关资源

- [SwiftUI Documentation](https://developer.apple.com/documentation/swiftui)
- [SwiftUI Tutorials](https://developer.apple.com/tutorials/swiftui)
- [WWDC SwiftUI Sessions](https://developer.apple.com/videos/all-videos/?q=swiftui) 