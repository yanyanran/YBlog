# HitTest底层原理

目前hitTest方法有三种组合技：

1、WidgetsBinding.instance.renderView.hitTest(result, position: tapPosition)

❌结果只能返回最底层的renderObject

此默认hitTest的原理是：
（1） 先判断事件触发位置是否位于组件范围内，如果不是则不会通过命中，hitTest 返回 false，如果是：
（2）先调用 hitTestChildren() 判断是否有子节点通过命中测试，是则将当前节点添加到 HitTestResult 列表，hitTest 返回 true。即只要有子节点通过了命中测试，那么它的父节点（当前节点）也会通过命中测试。如果没有子节点通过命中测试，则会取 hitTestSelf 方法的返回值，如果返回值为 true，则当前节点通过命中测试，反之否。

源码如下：

```
if (_size!.contains(position)) {
  if (hitTestChildren(result, position: position) || hitTestSelf(position)) {
    result.add(BoxHitTestEntry(this, position));
    return true;
  }
}
return false;
```

而默认的hitTestChildren和hitTestSelf的值都为false。所以在使用时必须重写hitTestChildren来确保成功遍历调用子节点的hitTest方法（重写hitTest也行）

因为hitTestChildren中会递归遍历子节点的hitTest方法，所以组件树的命中顺序是深度优先的，也就是说命中后子组件会比父组件优先加入到resList中。数组中的第一个元素是最上层的组件。


还有一个问题就是：开发过程中遇到重叠的组件，理想预期应该是最上层组件响应效果而非最底层。这就需要在hitTest时：
（1）使用倒序的遍历
（2）要么正序遍历时不要遇到命中的就返回，而是要遍历整一个树的所有子节点直到当前节点为叶子节点。（第二个方法）


2、
final renderObj = Globalkey.currentContext?.findRenderObject();
final resList = inn.hitTest(tapPosition, renderObj!);

☑️使用globalKey定位renderObject并重写hitTest。其中inn是一个自己实现的类，其中重写了hitTest方法

WidgetInspectorService.instance.selection

此方法有个缺点：不太通用，因为需要引入GlobalKey。


3、
final renderObj =  WidgetsBinding.instance.renderView;
final resList2 = inn.hitTest(tapPosition, renderObj);

组合技，使用widgetBinding的renderView，但hitTest用自己写的。
❌但是结果不对，像是只有局部选中了，待排查。

☑️应该使用WidgetsBinding.instance.renderView的child，就可以实现了。



