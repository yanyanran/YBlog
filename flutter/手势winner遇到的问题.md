Winner、stackInfo => 

在acceptGesture之前的debugWInner中直接获取



Owner、createLocation => 

从entry中获取



❌问题：滑动、缩放手势的winner.debugOwner为空，因为有些自定义手势的debugOwner开发者不一定会写，所以是获取不到widget的，但是如果从hitTestResult中取的话不一定是正确的。

断点调试后堆栈显示，每个手势在添加到竞技场前都有经过手势分发，也就是dispatchEvent，而在dispatchEvent中可以拿到HitTestEntry，由于handleEvent之后这个HitTestEntry就被丢弃了，所以咱拿不到。

![img](https://github.com/yanyanran/pictures/blob/main/Pasted%20Graphic%201.png?raw=true)

所以每次在这里保存一下即可，等到add方法时就可以取到了。相当于将手势和对应的renderobj关联起来。

![img](https://github.com/yanyanran/pictures/blob/main/Pasted%20Graphic%202.png?raw=true)

这样就可以成功拿到手势关联的控件和控件创建位置了（renderObject的x y和element的width、height同理）：

![img](https://github.com/yanyanran/pictures/blob/main/Pasted%20Graphic%203.png?raw=true)