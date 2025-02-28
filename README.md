# iOS 

本项目整理了 iOS 的高频考点和重要知识点。每个主题都包含详细的技术分析和实践经验。

## 目录

### 1. Runtime 机制
- [消息发送和转发机制](./Runtime/message-forwarding.md)
- [Method Swizzling 原理](./Runtime/method-swizzling.md)
- [Category 实现原理](./Runtime/category.md)
- [Associated Object](./Runtime/associated-object.md)
- [Class 与 Meta Class](./Runtime/class-meta-class.md)

### 2. 内存管理
- [ARC 实现原理](./Memory/arc.md)
- [循环引用问题](./Memory/retain-cycle.md)
- [AutoreleasePool](./Memory/autorelease-pool.md)
- [Tagged Pointer](./Memory/tagged-pointer.md)
- [Weak 实现原理](./Memory/weak.md)

### 3. 多线程与并发
- [GCD 详解](./Multithreading/gcd.md)
- [NSOperation](./Multithreading/nsoperation.md)
- [线程安全](./Multithreading/thread-safety.md)
- [锁机制](./Multithreading/lock.md)
- [线程通信](./Multithreading/thread-communication.md)

### 4. 性能优化
- [启动优化](./Performance/launch-optimization.md)
- [内存优化](./Performance/memory-optimization.md)
- [卡顿优化](./Performance/ui-optimization.md)
- [包体积优化](./Performance/package-optimization.md)
- [耗电优化](./Performance/battery-optimization.md)

### 5. 架构设计
- [架构模式对比](./Architecture/architecture-patterns.md)
- [组件化方案](./Architecture/componentization.md)
- [路由设计](./Architecture/router.md)
- [设计模式](./Architecture/design-patterns.md)
- [模块解耦](./Architecture/decoupling.md)

### 6. 网络相关
- [网络协议](./Network/protocols.md)
- [网络优化](./Network/optimization.md)
- [抓包原理](./Network/packet-capture.md)
- [网络安全](./Network/security.md)
- [框架源码分析](./Network/frameworks.md)

### 7. 系统框架
- [UIKit 优化](./Framework/uikit.md)
- [Core Animation](./Framework/core-animation.md)
- [Core Graphics](./Framework/core-graphics.md)
- [Block 原理](./Framework/block.md)
- [KVO/KVC 原理](./Framework/kvo-kvc.md)

### 8. 跨平台与新技术
- [Flutter](./Cross-Platform/flutter.md)
- [React Native](./Cross-Platform/react-native.md)
- [SwiftUI](./Cross-Platform/swiftui.md)
- [混编优化](./Cross-Platform/mixed-development.md)
- [热修复方案](./Cross-Platform/hot-fix.md)

## 如何使用本指南

1. 建议按照目录顺序进行学习
2. 每个知识点都包含：
   - 基本原理
   - 源码分析
   - 实践案例
   - 面试要点
3. 重点关注带有 🔥 标记的内容，这些是面试的重中之重

## 贡献指南

欢迎提交 Pull Request 来完善本指南。提交时请确保：
1. 内容准确性
2. 代码示例完整且能运行
3. 包含实际工作中的案例 