# 手势winner的预期效果

一个可拖拉悬浮按钮开关，点击后开启一个控制面板，上面有对应开关。开关开后，点击屏幕任意位置就能识别手势winner并给它的所属控件添加一个遮罩，长按遮罩会出现一个全屏展示页展示winner信息、代码创建位置还有堆栈信息之类的，多出页面的信息可以下划查看，再次点击就能关闭。

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