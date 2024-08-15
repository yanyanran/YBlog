# 手势winner的预期效果

一个可拖拉悬浮按钮开关，点击后开启一个控制面板，上面有对应开关。开关开后，点击屏幕任意位置就能识别手势winner并给它的所属控件添加一个遮罩，长按遮罩会出现一个全屏展示页展示winner信息、代码创建位置还有堆栈信息之类的，多出页面的信息可以下划查看，再次点击就能关闭。



# 实现原理

主要对Flutter手势事件的处理流程下手，在讲实现原理前先概括一下整个手势处理和竞争的过程：手势down事件时通过hitTest()确定可能触发手势事件的Widget并给它们设置优先级，再调用dispatchEvent对手势事件进行分发，并向手势竞技场Arena注册成员member等待判定。在handleEvent()时根据条件解决冲突判定手势的胜利者。

基于手势分发和处理流程，在每次acceptGesture宣告手势胜利之前即可捕获透传GestureArenaMember的信息，包括手势winner、所属控件、堆栈信息以及代码创建位置。



Winner、stackInfo => 

在acceptGesture之前的debugWInner中直接获取



Owner、createLocation => 

从entry中获取



# 遇到的问题

1、

**控件的位置信息拿不到。通过手势竞技场的winner拿到关联组件的widget，可以拿到宽高信息，但是距离遮罩绘制这块还需要知道offset，这个只能通过renderObeject去取。现在的问题是拿到了widget但是拿不到关联的element**

（尝试在处理手势事件之前的hitTest中去将事件和对应的renderObject关联起来）❌没必要那么麻烦：

_handlePointerEventImmediately

=> hitTestInView :

flutter/lib/src/rendering/binding.dart

**原理**：在设置IgnorePointer => ignoring=true后，将会在hitTest 中直接判断识别，ignore的组件将不会添加到hitTestResult中

所以flutter-sdk中直接取得_*handlePointerEventImmediately*在hitTestInView之后存的的*debugHitTestResList ==>* GestureBinding.*debugHitTestResList*

取到[*0*]和[*len-3*]也就是第一个和倒数第二个*renderObj*，取差值即可找到*left*、*right*、*bottom*和*top*，就可以将*widget*正确绘制出来了



2、

close竞技场的时候会有两种情况：一种是竞技场内只有一个成员，一种是有多个成员并且state.eagerWinner 不为空，此时说明竞技场中有的member优先级高，直接宣告胜利。

**在情况一的时候，宣布手势获胜之前调用我写的_debugWinner函数会使得SingleChildScrollView组件响应不了滚动效果。**

是滑动手势也触发了手势竞争，调用了debug函数，在取获胜手势的debug Owner的时候取得空值。在GestureArenaManager里添加了一个debug开关并对外暴露一个接口，当debug模式开启时才调用该函数，并且在取手势winner的时候对debugOwner做一个判空。



3、



❌问题：滑动、缩放手势的winner.debugOwner为空，因为有些自定义手势的debugOwner开发者不一定会写，所以是获取不到widget的，但是如果从hitTestResult中取的话不一定是正确的。

断点调试后堆栈显示，每个手势在添加到竞技场前都有经过手势分发，也就是dispatchEvent，而在dispatchEvent中可以拿到HitTestEntry，由于handleEvent之后这个HitTestEntry就被丢弃了，所以咱拿不到。

![img](https://github.com/yanyanran/pictures/blob/main/Pasted%20Graphic%201.png?raw=true)

所以每次在这里保存一下即可，等到add方法时就可以取到了。相当于将手势和对应的renderobj关联起来。

![img](https://github.com/yanyanran/pictures/blob/main/Pasted%20Graphic%202.png?raw=true)

这样就可以成功拿到手势关联的控件和控件创建位置了（renderObject的x y和element的width、height同理）：

![img](https://github.com/yanyanran/pictures/blob/main/Pasted%20Graphic%203.png?raw=true)