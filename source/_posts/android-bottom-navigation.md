---
title: android 底部导航栏规范
date: 2016-04-25 14:09:21
updated: 2016-04-25 14:09:21
tags:
categories:
---

- 最后更新: 2016-04-27
- 修改记录:
	* 2016-04-25 初稿
	* 2016-04-27 增加截图

---

# 0. 前言：
最近 Google 在 Material Design 设计规范中加入底部导航栏（Bottom navigation）设计（参考文档: [Components - Bottom navigation](https://www.google.com/design/spec/components/bottom-navigation.html) ），现在我们先来学习 Bottom navigation 在设计、使用、交互、风格、尺寸的一些规范。水平有限，如理解有误，请多多吐槽。

 ![](/images/14615646982872.png)

 
# 1. 简单介绍
底部导航栏（Bottom navigation）这种设计最初是参考了 iOS 的设计规范，Google 在 Android 的设计规范上其实一直不推荐采用的。但是，很多 Android App 中都是随处可见的，如支付宝、微信、QQ等都使用了，它允许用户可以在多个顶级视图之间快速切换。

### 注意
 底部导航栏推荐在手机上使用，平板或者桌面系统还是推荐侧边的导航栏
 
 * 移动设备
 ![](/images/14617332106360.png)

* 较大显示设备，比如平板，桌面应用
 ![](/images/14617332508427.png)

 

# 2. 如何使用
（1）底部导航栏需要有 3-5 个标签（tab）,并且每个tab选择的视图重要性要相似，对于少于 3个 tab 的情况，是不推荐使用 Bottom navigation 的。

* 正确方式：
![](/images/14615652601150.png)


* 错误方式：
 ![](/images/14615652891922.png)

 
（2）如果标签很多，比如有超过了 5 个这种情况呢？Google 也是不提倡使用 Bottom navigation 的，可以用 Drawer navigation 替换。
 
* 正确方式：
 ![](/images/14615653042210.png)


 
* 错误方式：
 ![](/images/14615653215230.png)


  
# 3. 风格样式
（1）标签 Icons 和文字的颜色选择是很重要的，一亮一暗才能有对比，用户才很快知道你选择了哪个，如果五颜六色，你是很难分清选择了哪个的。

	* 当前标签是选中状态的时候，显示图标和文字。
	* 当导航栏只有三个标签的时候，他们的图标和文字都应该被显示。
	* 当导航栏有四个或者五个标签的时候，在非选中状态的时候只显示他们的图标即可。
	* 导航栏选中的标签使用应用的主色来填充图标和文字
	* 如果导航栏有背景色，使用白色来填充图标和文字






* 正确方式：
![](/images/14615653768533.png)

![](/images/14617350719576.png)


* 错误方式：
![](/images/14615653979084.png)

![](/images/14617350867144.png)

 
 （2）标签的文字说明要简短而有意义，避免太长的，也不提倡太长了换行和省略的方式

* 正确方式：
![](/images/14615654161493.png)


 
* 错误方式：
![](/images/14616016444449.png)

![](/images/14616016778061.png)

 ![](/images/14616017091194.png)


    

## 注意：
* 1. 选中的标签要展示高亮图标和文字
* 2. 如果是3个标签的时候，要展示 Bottom navigation bar 中所有的图标和文字
* 3. 如果是多于 3 个标签的情况，没选中的 tab 只需要展示图标就可以，不用文字说明

    
# 4. 行为交互
（1）用户上拉列表时，隐藏Bottom navigation，下拉列表时，显示Bottom navigation
![](/images/14616017631338.gif)



(2)点击Bottom navigation Icon 的时候，不能打开菜单选择或者其他弹窗操作，而只是刷新当前视图的内容，如下图：

![](/images/14616017928800.gif)


(3)不推荐使用手势在视图内容区域切换视图

正确方式：
![](/images/14616018078094.gif)


错误方式：
![](/images/14616018251796.gif)

# 5. 固定风格导航栏尺寸设计
（1）Bottom navigation对于尺寸的要求还是挺严格的，标签选中和没选中都有细微的差别。
![](/images/14616018437232.png)

效果如下：

![](/images/14616018655355.gif)

（2）具体的一些尺寸：

* 最大宽度： 168 dp
* 最小宽度： 120 dp（大屏幕设备），104 dp（小屏幕设备）
* 高度： 56 dp
* 图标： 24*24 dp
* 内容对齐模式： 文字和图片水平居中
* 边距（padding）： 
	* 选中的标签 ，图片上边距 6 dp；未选中标签，图片上边距 8 dp
	* 文字下边距 10 dp
	* 文字左右各 12 dp
* 文字：
	* 选中标签 Roboto Regular 14sp
	* 未选中标签 Roboto Regular 12sp
	
移动设备横屏效果	
	 ![](/images/14617342550476.png)

平板效果
![](/images/14617343097033.png)

# 6. 切换风格导航栏尺寸设计
（1）选中和未选中的效果更明显
![](/images/14617355603306.png)

效果:
![](/images/14617377457930.gif)

(2) 尺寸规范:

* 选中标签
	* 最大宽度：168 dp
	* 最小宽度： 96 dp	
* 未选中标签
	* 最大宽度： 96 dp
	* 最小宽度： 64 dp
* 高度： 56 dp
* 图标： 24*24 dp
* 内容对齐模式： 文字和图片水平居中
* 边距（padding）： 
	* 选中的标签 ，图片上边距 6 dp；未选中标签，图片上边距 16 dp
	* 下边距 10 dp
* 文字：Roboto Regular 14sp

移动设备横屏模式：
![](/images/14617380707923.png)

平板模式：
![](/images/14617380840138.png)




# 7. 杂谈
Android官网中有这么一句话：

[Don't use bottom tab bars](http://developer.android.com/design/patterns/pure-android.html)

Other platforms use the bottom tab bar to switch between the app's views. Per platform convention, Android's tabs for view control are shown in action bars at the top of the screen instead. In addition, Android apps may use a bottom bar to display actions on a split action bar.

You should follow this guideline to create a consistent experience with other apps on the Android platform and to avoid confusion between actions and view switching on Android.

![](/images/14617351763741.png)



之前说好的Meterial Design 规范说变就变，自己都不遵守，有何规范可言。

Google+ 截图
![](/images/14617385715582.jpg)


然而，Google +已经加入了底部导航栏（Bottom navigation），打脸归打脸，决定这一因素的还是用户。

我们说说 Drawer navigation，不要觉得疑惑，关 Drawer 什么事？没错，就是和它有关，认真想想，使用 Drawer 切换视图时，使用方式是点击按钮或者手势滑动打开，然后才选择跳转的标签，虽然节省了手机视图空间，但是操作较不方便以致使用率低，从而降低其他页面的跳转率这个缺点是不能忽视的。
所以说，相对于 Drawer navigation , Bottom navigation 就有优势了。

然后，对于我们开发来说，又是一大难点了。

比如 FB、SnackBar 等显示的方式，个人觉得都可以不需要 FB 了，至于SnackBar，看下图：

![](/images/14617352823186.png)



这这这，原谅我欣赏不了这种美。

# 总结：被坑的最惨的还是 Android 程序员！
























