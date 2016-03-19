---
title: Android 日志的优化实践
date: 2015-11-04 17:16:19
updated: 2015-11-14 17:16:19
tags:
- android
- log
categories:
- android
---

- 最后更新: 2015-11-14
- 修改记录:
	* 2015-11-04 初稿
	* 2015-11-10 增加日志需求
	* 2015-11-14 增加实现细节


## 1. 简介
在日常的 Android 开发中，日志打印是一项必不可少的操作，我们通过分析打印的日志可以分析程序的运行数据和情况。本文主要介绍了如何更好的使用日志。先来介绍下相关的原理，然后来说说怎么优化开发体验。

## 2. 哪些形式
* System.out.println
	这是标准的Java输出方法，相信很多公司都不提倡使用，这里进行列举，目的是为了提醒大家不用。

* Android Log
	Android 自身提供了一个日志工具类，那就是 `android.util.Log`。使用很简单，如下：

```java
Log.i(LOGTAG, "onCreate");
```

我们在调试的时候经常会输出这行代码，这个方法有两个参数，一个是TAG，一个是真正要输出的内容。

## 3. TAG 的选取
TAG 选择虽然没有强制要求，一般来说只要是个字符串就行，但是好的 TAG 策略，可以提高开发体验。

### 3.1 选用人名

很多人都曾采用人名的形式，比如:
	
```java
	Log.i("lilei", "onCreate");
```

这样做的目标一是为了过滤方便，当一个人在写一个模块多个文件时，使用这个形式，过滤起来很容易帮助理解程序的执行情况。另外的目的就是为了表明日志周围代码的作者姓甚名谁。

然而，我却不推荐这种人名作为TAG的形式。原因如下：

* 以人名作为关键字过滤，不易确定产生日志的类文件
* 随着某个人模块实现的增加，过滤人名易产生来自其他模块的干扰信息。

### 3.2 动态选取

还有一种选取方式，就是：

```java
	private static final String LOGTAG = Settings.class.getSimpleName();
```

这样使用，得到的 `LOGTAG` 的值就是 `Settings`。这么做，开发版使用的时候非常方便；但是当应用打包发布时，`Settings` 这个类进行了混淆之后，类名变成了类似 a,b,c 这样的名称，`LOGTAG` 则不再是 `Settings` 这个值了。这样可能造成的问题就是，内部混淆有日志的包，我们无法再通过过滤 `Settings` 来得到相应的信息。

### 3.3 静态的 tag

一般比较推荐的形式就是以字符串字面量形式去设置 LOGTAG。如下，在`Settings`类中

```java
	private static final String LOGTAG = "Settings";
```

## 4. 屏蔽日志输出
主要是通过两种方式实现的，一个是在编译期时候屏蔽；一个是在运行时中处理。

### 4.1 运行时屏蔽
通常做法，是自定义一个类

```java
public class L {
    private static final boolean ENABLE_LOG = true;

    public static void i(String tag, String message) {
        if (ENABLE_LOG) {
            android.util.Log.i(tag, message);
        }
    }
}
```

通过调用 L.i 来输出日志,通过修改 `ENABLE_LOG` 的值可以方便的切换是否屏蔽日志输出。

**注意**

```java
Log.i(LOGTAG, "sdkVersion="+getVersion());
```
虽然可以屏蔽日志输出，但是并不会屏蔽语句中的代码 `getVersion` 执行，另外字符串的拼接也会被执行到。

### 4.2 编译期屏蔽
主要是利用 `Proguard` 在代码混淆时，去掉相关的日志代码。

```java
-assumenosideeffects class com.logtest.L {
        public static *** i(...);
}
```
可以用同样的方式来处理原生的日志打印

```java
-assumenosideeffects class android.util.Log {
        public static *** d(...);
        public static *** e(...);
        public static *** i(...);
        public static *** v(...);
        public static *** println(...);
        public static *** w(...);
        public static *** wtf(...);
}
```

下面来个栗子,看看反编译的出来的代码:

先来个测试的代码，其他无关部分都先忽略掉：

```java
public class MainActivity extends Activity {
    private static final String LOGTAG = "MainActivity" ;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        dumpDebugInfo();
    }

    private void dumpDebugInfo() {
        Locale defaultLocale = Locale.getDefault();
        Log.i(LOGTAG, "sdkVersion=" + Build.VERSION.SDK_INT + "; Locale=" + defaultLocale);
    }

}
```
然后使用 `proguard` 的配置

```java
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** i(...);
    public static *** v(...);
}

-assumenosideeffects class com.droidyue.logdemo.DroidLog {
        public static *** i(...);
}
```

生成 apk 之后,反编译,得到 smali 源码:

```java
# virtual methods
.method protected onCreate(Landroid/os/Bundle;)V
    .locals 3

    invoke-super {p0, p1}, Landroid/app/Activity;->onCreate(Landroid/os/Bundle;)V

    const v0, 0x7f030017

    invoke-virtual {p0, v0}, Lcom/test/logdemo/MainActivity;->setContentView(I)V

    invoke-static {}, Ljava/util/Locale;->getDefault()Ljava/util/Locale;

    move-result-object v0

    new-instance v1, Ljava/lang/StringBuilder;

    const-string v2, "sdkVersion="

    invoke-direct {v1, v2}, Ljava/lang/StringBuilder;-><init>(Ljava/lang/String;)V

    sget v2, Landroid/os/Build$VERSION;->SDK_INT:I

    invoke-virtual {v1, v2}, Ljava/lang/StringBuilder;->append(I)Ljava/lang/StringBuilder;

    move-result-object v1

    const-string v2, "; Locale="

    invoke-virtual {v1, v2}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    move-result-object v1

    invoke-virtual {v1, v0}, Ljava/lang/StringBuilder;->append(Ljava/lang/Object;)Ljava/lang/StringBuilder;

    move-result-object v0

    invoke-virtual {v0}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    return-void
.end method
```

无论是运行时日志屏蔽还是编译期，message参数上发生的字符串拼接都依然存在。但是编译期屏蔽减少了方法调用（即方法进出栈操作）.


## 5. 优化实践
本来文章到这里就可以结束了。介绍了原理，接下来小伙伴们就可以愉快的编码了。但是作为有追求的理想青年，我们还是可以利用 log 来提升开发体验的。

### 对于一个需要重复无数遍的函数，我们需要尽可能的简化

在应用开发中，我们可能需要无数次的写下:

```java
	Log.d(TAG, "This is a debug log");
```
很多时候，我们仅需要个输出，于是就有了第一个需求，省略 TAG，或者说，我们希望有一个东西可以帮我们定义好TAG，我们只需要写正真有意义的内容就行。

更进一步，虽然现在 IDE 都有了代码自动提示，但是能少打几个字总是好的。我们希望 `L` 来替代 `Log`
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474252600535.jpg)

### 然后呢，我们希望 log 输出的更加美观。最好输出的有一个 `超链接` ，点击就能跳转到相应的代码中
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474254548953.jpg)

 ==这个功能只支持 Android Studio，Eclipse只能输出,无法跳转==

### 有时候，我们需要的只是输出一个简单的字符，有时候，我们又希望能把函数调用的堆栈也打印出来。
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474270903687.jpg)
或者是
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474271902206.jpg)



### 最好能支持数据的可视化，而不用手动去拼接字符串或者简单的通过 `Object.toString()` 来打印对象信息。

* json 格式化
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474166252179.jpg)

* 数组等数据结构
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474294418591.jpg)
对应输出
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474294935656.jpg)

* 其他对象
![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474295884906.jpg)




### 哦，还有。加个能控制 log的开关,另外能够方便的定制 log 的输出,比如输出到文件或者回传服务器


## 6. 实现
上面的需求实现的难度并不是很大，这里就大致介绍下实现的思路。

![](http://7xqfqq.com1.z0.glb.clouddn.com/2016-03-19-14474327190712.jpg)

### 获取当前类名

``` java
private static String getClassName() {        String result;        StackTraceElement thisMethodStack = (new Exception()).getStackTrace()[2];        result = thisMethodStack.getClassName();        int lastIndex = result.lastIndexOf(".");        result = result.substring(lastIndex + 1, result.length());        //如果调用位置在匿名内部类的话，就会产生类似于 MainActivity$3 这样的TAG        //可以把$后面的部分去掉        int i = result.indexOf("$");        return i == -1 ? result : result.substring(0, i);    }
```

### 输出函数文件信息

``` java
/**     * log这个方法就可以显示超链     */    private static String callMethodAndLine() {        String result = "at ";        StackTraceElement thisMethodStack = (new Exception()).getStackTrace()[1];        result += thisMethodStack.getClassName()+ ".";        result += thisMethodStack.getMethodName();        result += "(" + thisMethodStack.getFileName();        result += ":" + thisMethodStack.getLineNumber() + ")  ";        return result;    }
```
  
### 数据可视化实现
* `array`,`map`,`set` 等容器可以通过遍历显示
* `Object` 可以通过反射方式,拿到变量以及变量的类型
* `xml`,`json` 可以通过自定义格式化来实现


## 7. 后记
 通过这篇文章，给大家介绍了一个日志输出优化的过程。通过实践，说明一个最简单的日志输出都是有很多可以优化的地方；有时候，一些小小的改造，可以极大的提升开发的体验。
 
 最后的最后，你可能不太喜欢这样的日志输出。没关系，相信我们的最终目的是一致的，就是让开发越来越便捷，代码越来越优雅~
 




