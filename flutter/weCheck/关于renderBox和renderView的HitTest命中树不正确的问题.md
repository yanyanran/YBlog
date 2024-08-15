关于renderBox和renderView的HitTest命中树不正确的问题

- WidgetsBinding.instance.renderView.hitTest()

这个方法是在flutter的**root渲染对象**（renderView）上执行命中，用于全局命中。

- final box = key.currentContext?.findRenderObject() as RenderBox;              box.hitTest(BoxhitTestResult, position: tapPosition);
  => RenderBox.hitTest

这个方法是在**特定**的RenderBox上执行命中，用于局部命中。（在特定的控件比如按钮或列表项上捕获用户点击并确定与哪个子渲染对象相交）