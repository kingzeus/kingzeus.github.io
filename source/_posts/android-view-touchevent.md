---
title: Android View 的事件机制详解
date: 2016-04-13 20:25:52
updated: 2016-04-13 20:25:52
tags:
- android
- view
- touchEvent
categories:
- android
---

- 最后更新: 2016-04-13
- 修改记录:
	* 2014-04-13 初稿
	* 2014-04-18 补充示例

## 1. View 事件分发机制
View 是 Android UI 的一个基础，而事件分发机制是 View 的一个核心知识点更是一个难点，很多同学对这个问题都会比较困惑。而 View 的另一个难点滑动冲突的解决也依赖于事件分发机制。

### 1.1 MotionEvent
MotionEvent 用来描述用户触发的屏幕动作的事件。

当你的一个手指在屏幕上滑动一下时，系统会产生一系列的触摸事件对象。典型的事件包括：

* **ACTION_DOWN** = 0 —— 手指触摸到屏幕
* **ACTION_UP** = 1 —— 手指从屏幕上移开
* **ACTION_MOVE** = 2 —— 手指在屏幕上移动
* **ACTION_CANCEL** = 3 —— 动作取消
* **ACTION_OUTSIDE** = 4 —— 动作超出边界
* **ACTION_POINTER_DOWN** = 5 —— 已有一个点被按住，再按下另外一个点
* **ACTION_POINTER_UP** = 6 —— 多点被按下的时候，非最后一个点抬起

![](/images/14605560667360.gif)

正常情况下，一次手指触摸屏幕可能产生的事件：

* 按下后直接松开  序列为： DOWN -> UP
* 按下后移动一阵之后在松开 序列为： DOWN -> MOVE ... -> MOVE -> UP

同时我们可以通过 MotionEvent 得到点击的 坐标。为此，提供了 2组 函数 `getX/getY` 和 `getRawX/getRawY` 。区别在于：

*  `getX/getY` 返回相对于当前 View 左上角的坐标
*  `getRawX/getRawY` 返回相当于屏幕左上角的坐标

### 1.2 TouchSlop
TouchSlop 是系统能识别出来的被认为是滑动的最小距离。也就是说，手指在屏幕上滑动时，如果两次滑动的距离小于这个常量，那么就不会被识别成滑动。

TouchSlop 可以用来防止按键位置的抖动。这个常量是设备相关的，可以通过 `ViewConfiguration.getScaledTouchSlop()` 来获取。

## 2. 点击事件的相关处理函数
一个点击事件 (MotionEvent) 产生以后，系统需要把这个事件传递给一个具体的 View，而这个传递的过程就是事件的分发过程。点击事件的分发过程由 3 个方法来共同完成：

* public boolean dispatchTouchEvent (MotionEvent ev)

用来进行事件分发。如果事件能够传递给当前的 View ，那么这个方法就一定会被调用，返回结果受当前 View 的 onTouchEvent 和 子 View 的 `dispatchTouchEvent` 的影响。返回事件是否被消费。

* public boolean onInterceptTouchEvent (MotionEvent ev)

在 `dispatchTouchEvent` 内部调用，用来判断是否拦截某个事件。返回是否拦截事件。

* public boolean onTouchEvent(MotionEvent ev)

在 `dispatchTouchEvent` 内部调用，用来处理点击事件，返回结果表示是否消费了当前事件。

## 3. 举个栗子

* 下面是一个自定义的 Activity

![](/images/14609483250389.png)

* 布局简略关系

![](/images/14609484458874.png)

* 事件分发

下图显示了，没有拦截时候的事件分发顺序

	手指在 View1 上操作时事件分发顺序
	当按下事件没有被拦截，那么所有状态的事件都由Activity进行处理
	
![](/images/14609487247618.png)
	

* 事件消费
通过 dispatchTouchEvent 对事件进行处理，当返回值为 true 的时候表示消费了事件。

		ViewGroup1 中的 dispatchTouchEvent 直接返回 true
		手指在 View1 上操作时事件分发顺序
		事件传递到 ViewGroup1 后被消费，后续事件没有分发给子控件

![](/images/14609490327325.png)

* 事件拦截
通过 onInterceptTouchEvent 拦截事件，当返回值为 true 的时候拦截事件

		ViewGroup2 中的 onInterceptTouchEvent 直接返回 true
		手指在 View1 上操作时事件分发顺序
		事件传递到 ViewGroup2 后被拦截，不会再分发给子控件

![](/images/14609602211963.png)

* 事件处理
当 onTouchEvent 返回 true ，表示事件被当前控件消费

		ViewGroup2 中的 onInterceptTouchEvent 直接返回 true
	  	ViewGroup2 中的 onTouchEvent 方法中添加 按下事件 返回 true
		当手指对View1点击、移动、抬起时
		事件传递到 ViewGroup2 后被拦截，后续事件先发送给 ViewGroup2 处理，然后返回 Activity 处理


![](/images/14609607629304.png)


		ViewGroup2 中的 onInterceptTouchEvent 直接返回 true
	  	ViewGroup2 中的 onTouchEvent 方法中 直接返回 true
		当手指对View1点击、移动、抬起时
		事件传递到 ViewGroup2 后被拦截，后续事件全部由 ViewGroup2 处理

![](/images/14609608813851.png)


* 下面来看下 button 的处理

		用 Button1 替换 View1
		其余函数使用默认实现
		button 默认就是直接截获和消费了事件
	
![](/images/14609611313192.png)

	View 的 *clickable* 属性的效果，就如同 button 一样。
	
## 4. dispatchTouchEvent 伪代码
![](/images/14609640887128.jpg)

* 对于一个根 ViewGroup 来说，点击事件产生以后，会首先传递给它，此时 它的 dispatchTouchEvent 就会被调用
* 如果这个 ViewGroup 的 onInterceptTouchEvent 返回 true 就表示它要拦截事件，那么接下去的事件都会由这个 ViewGroup 处理，即 onTouchEvent 会被调用
* 如果这个 ViewGroup 的 onInterceptTouchEvent 返回 false 就表示它不会拦截事件，当前事件就会传递给子控件，接着调用子控件的  dispatchTouchEvent
* 如此反复直到事件被最终处理
* 当一个 View 需要处理事件时，如果设置了 onTouchListener，则优先处理 onTouchListener 的 onTouch 事件，根据 onTouch 的返回值，来决定 onTouchEvent 是否被调用（true 表示消费了事件，不会再调用了 onTouchEvent）。如果设置了 onClickListener ，则会在 onTouchEvent 中被调用。
* 如果 View 的 onTouchEvent 返回 false，即没有消费事件，就会调用父控件的 obTouchEvent ，直到 Activity 的 obTouchEvent。

## 5. 事件传递机制的结论
1. 同一个事件序列是指从手指接触屏幕的那一刻开始，直到手指离开屏幕的那一刻结束。在这个过程中产生了一系列的事件合集。这个事件序列从 DOWN 开始，中间含有数量不等的 MOVE 事件，最终以 UP 事件结束。
2. 正常情况下，一个事件序列只能被一个 View 拦截且消耗。
3. 某个 View 一旦决定拦截事件，那么这个事件序列都由它来处理，并且它的 onInterceptTouchEvent 不会再被调用，即后续事件不会再去询问是否要拦截。
4. 某个 View 一旦开始处理事件，如果它不消费 ACTION_DOWN 事件(onTouchEvent 返回 false)，则后续的事件不会交给它来处理，事件会交由它的父控件来处理（父控件的 onTouchEvent 会被调用）。
5. 如果 View 不消费除 ACTION_DOWN 以外的事件，那么这个点击事件会消失，此时父控件的 onTouchEvent 并不会被调用，并且当前 View 能收到后续事件，最终这些事件会交由 Activity 来处理。
6. ViewGroup 默认不拦截事件，onInterceptTouchEvent 默认返回 false。
7. View 没有 onInterceptTouchEvent ，一旦有事件到达，那么 onTouchEvent 就会被调用。
8. View 默认会消费事件，即 onTouchEvent 返回 true，除非它是不可点击（clickable 和 longClickable 同时为 false），其中 longClickable 默认为 false；clickable 则需要根据控件讨论，button 默认返回 true；textView 默认返回 false
9. View 的 enable 属性不影响 onTouchEvent 的默认返回值。
10. onClick 发生的前提是当前 View 可点击，且收到了 down 和 up 事件。
11. 事件传递总是从父控件传递到子控件，子控件可以通过 requestDisallowInterceptTouchEvent 来干预父控件的事件分发，但是 ACTION_DOWN 除外。


## 6. 参考
* [公共技术点之 View 事件传递](http://p.codekk.com/blogs/detail/54cfab086c4761e5001b253e)
* [View的事件分发机制](http://www.jianshu.com/p/49d4043621d6)


