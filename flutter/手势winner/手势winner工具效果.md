## 开发背景-手势冲突

本工具主要在于解决交互设计中的难题。



### 手势冲突的原因

1、多点触控设备的普及使得用户可以更自由地进行手势操作，但同时也增加了手势冲突的可能性。
2、应用程序的复杂度不断提高，功能越来越丰富，导致手势操作的复杂性增加。
3、设计师对于手势操作的理解和掌握程度不同，容易产生手势冲突。



### 效果视频

基于flutter手势处理基础上实现的捕获胜出手势效果如下：

https://github.com/yanyanran/pictures/blob/main/panel.mp4



## 效果展示

开启工具后用户下发手势（双击、点击、长按等），当前屏幕上将会绘制出响应该手势的组件区域。长按区域即可展示当前的胜出手势信息、手势响应所属控件以及其代码创建位置，以及当前的堆栈信息。

##### 例1

当前手势点击

![](https://github.com/yanyanran/pictures/blob/main/tap1.jpg?raw=true)

长按后显示信息

![](https://github.com/yanyanran/pictures/blob/main/tap2.jpg?raw=true)

##### 例2

当前手势下拉

![](https://github.com/yanyanran/pictures/blob/main/long1.jpg?raw=true)

长按后显示信息

![](https://github.com/yanyanran/pictures/blob/main/long2.jpg?raw=true)

