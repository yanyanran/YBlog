## 背景

WeCheck工具主要用于产品设计阶段，在审查和绘制软件页面结构时，帮助设计师准确确定页面中各个组件的位置坐标、尺寸、颜色等属性。这便于设计师在审查过程中确保页面设计符合预期。本次开发重点是将WeCheck应用于微信Flutter页面，涵盖订阅号、听一听、图文、Skyline以及LiteApp等多种页面类型。

## 用法

### 启动手势

四指上划手势在当前页面唤醒WeCheck按钮，点击该按钮即可展开WeCheck功能菜单栏。

![img](https://github.com/yanyanran/pictures/blob/main/wecheck/%E8%8F%9C%E5%8D%95%E6%A0%8F.png?raw=true)

### 属性

用户通过点击即可选中包含当前点击点的最上层布局，并且看到具体的属性信息。

![img](https://github.com/yanyanran/pictures/blob/main/wecheck/%E5%B1%9E%E6%80%A7.png?raw=true)

### 间距

（1）单击一个布局，弹窗展示当前选中布局相对于整个屏幕的边距距离，点击自己取消选中。

![img](https://github.com/yanyanran/pictures/blob/main/wecheck/%E9%97%B4%E8%B7%9D1.png?raw=true)

（2）点击一个布局后点击另一个布局，将会展示两布局之间的间距

![img](https://github.com/yanyanran/pictures/blob/main/wecheck/%E9%97%B4%E8%B7%9D2.png?raw=true)

 

## 实现原理

Flutter层将主要处理函数和MethodChannel创建并监听，oc层负责绑定该channel并且传入用户的点击坐标。

##### 🎯Flutter层核心1 – HitTest的命中实现

hitTest方法的原生调用为**WidgetsBinding.*****instance*****.renderView.hitTest(result, position)**。直接使用时，实际上获取到的是最底层的renderObject。但是开发过程中遇到重叠布局的场景，我们理想预期应该是获取到最上层布局来响应效果，而非最底层。为什么只获取到了底层的？先看看默认的hitTest如何实现。

###### 默认核心源码

```dart
bool hitTest(HitTestResult result, { @required Offset position }) {
  ...  
  if (_size.contains(position)) { // 判断事件的触发位置是否位于组件范围内
    if (hitTestChildren(result, position: position) || hitTestSelf(position)) {// 默认的hitTestChildren和hitTestSelf值都为false
      result.add(BoxHitTestEntry(this, position));
      return true;
    }
  }
  return false;
}
```

hitTest的原理是先判断事件触发位置是否位于组件范围内：

1. 如果不是，则不会命中，hitTest 直接返回 false；
2. 如果是，则调用 hitTestChildren() 判断是否有子节点通过命中测试：

- 如果有，则将当前节点添加到 HitTestResult 列表，hitTest 返回 true。即只要有子节点通过了命中测试，那么它的父节点（当前节点）也会通过命中测试。
- 如果没有子节点通过命中测试，则会取 hitTestSelf 方法的返回值，如果返回值为 true，则当前节点通过命中测试，反之否。

因为hitTestChildren中会递归遍历子节点的hitTest方法，所以组件树的命中顺序是深度优先的，也就是说命中后子组件会比父组件优先加入到resList中。数组中的第一个元素就是最上层的布局。

*注意：默认的hitTestChildren和hitTestSelf的值都为false，所以我们在使用时**必须重写hitTestChildren或重写整个hitTest**来确保成功遍历调用子节点的hitTest方法。*

###### 重写代码逻辑

```dart
  List<RenderObject> hitTest(Offset position, RenderObject root) {
    final List<RenderObject> regularHits = <RenderObject>[];
    final List<RenderObject> edgeHits = <RenderObject>[];

    _hitTestHelper(regularHits, edgeHits, position, root, root.getTransformTo(null));
    // 计算RenderObject的命中区域大小
    double area(RenderObject object) {
      final Size size = object.semanticBounds.size;
      return size.width * size.height;
    }
    regularHits.sort((RenderObject a, RenderObject b) => area(a).compareTo(area(b))); // 对regularHits列表中的元素按照命中区域大小进行排序，面积越小越靠前
    final Set<RenderObject> hits = <RenderObject>{
      ...edgeHits,
      ...regularHits,
    };
    return hits.toList();
  }

// edgeHits 可以删除不加
  bool _hitTestHelper(
      List<RenderObject> hits,
      List<RenderObject> edgeHits,
      Offset position,
      RenderObject object,
      Matrix4 transform,
      ) {
    bool hit = false;
    final Matrix4? inverse = Matrix4.tryInvert(transform);
    if (inverse == null) {
      return false;
    }
    final Offset localPosition = MatrixUtils.transformPoint(inverse, position);

    final List<DiagnosticsNode> children = object.debugDescribeChildren();
    for (int i = children.length - 1; i >= 0; i -= 1) {
      final DiagnosticsNode diagnostics = children[i];
      if (diagnostics.style == DiagnosticsTreeStyle.offstage ||
          diagnostics.value is! RenderObject) {
        continue;
      }
      final RenderObject child = diagnostics.value! as RenderObject;
      final Rect? paintClip = object.describeApproximatePaintClip(child);
      if (paintClip != null && !paintClip.contains(localPosition)) {
        continue;
      }

      final Matrix4 childTransform = transform.clone();
      object.applyPaintTransform(child, childTransform);
      if (_hitTestHelper(hits, edgeHits, position, child, childTransform)) {  // 递归
        hit = true;
      }
    }

    final Rect bounds = object.semanticBounds;
    if (bounds.contains(localPosition)) {  // 检测是否在bounds里
      hit = true;  // 发生命中
      if (!bounds.deflate(_edgeHitMargin).contains(localPosition)) {  // 检查localPosition是否在缩小了_edgeHitMargin(2.0)的边界内
        edgeHits.add(object);  // localPosition不在缩小后的边界内，意味着它位于原始边界的边缘上
      }
    }
    if (hit) {
      hits.add(object);
    }
    return hit;
  }
```

1.  

从根节点进行递归获取到列表还要进过上层的一次排序，根据命中面积进行对比排序，最终面积越小的越靠前。

##### 🎯Flutter层核心2 – 渲染根节点RenderObject的选择

对于传入hitTest的遍历渲染树的根节点，我们有两个选择：

（1）**WidgetsBinding.*****instance*****.renderView.child**

（2）**Globalkey.currentContext.findRenderObject()**

两个实例出的renderObject传入到hitTest中都可以获取正确的结果，但是我们尽可能避免使用**Globalkey**。使用(2)方法需要在skyline、liteapp和home route下分别建立GlobalKey，这在一个大型项目中，对于状态管理是不利的，所以我们传入的是第一个对象。

 

⚠️注意：传入的不能是WidgetsBinding.instance.renderView，而必须是他的孩子节点child。