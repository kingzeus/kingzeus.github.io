---
title: android_ipc
tags:
categories:
---

From: <http://www.jianshu.com/p/2ab103e32456?utm_campaign=hugo&utm_medium=reader_share&utm_content=note&utm_source=weibo>

* * *

一个读书的小**Tips**：如果对原书内容不是很了解的小伙伴，可以先看读书笔记，心中有个**概要**，然后再细读原书； 

* * *

**学习内容**：开篇介绍了Android中的多进程概念已经多进程开发中常见的注意事项，接着介绍了Android序列化和Binder，以及详细介绍了Bundle、文件共享、AIDL、Messenger、ContentProvider和Socket这几种进程间通讯；还讲解了如何使用Binder连接池来达到多个AIDL共用一个Service。最后讲解了各种进程间通讯的优缺点和适用场景。接下来 咱们开始吧 

* * *

#### 2.1 Android IPC简介 

  * IPC为进程间通讯，两个进程之间进行数据交换的过程。
  * IPC不是Android所独有的，任何一个操作系统都有对应的IPC机制。Windows上通过剪切板、管道、油槽等进行进程间通讯。Linux上通过命名空间、共享内容、信号量等进行进程间通讯。Android中没有完全继承于Linux，有特色的进程间通讯方式是Binder，还支持Socket。
  * 使用场景：由于某些原因应用自身需要采用多进程模式来实现，或者为了加大一个应用可使用的内存，因为Android对当个应用可使用的最大内存做了限制。

* * *

#### 2.2 Android中的多进程模式 

  * 同一个应用，通过给四大组件指定android:process属性，就可以开启多进程模式。
  * 进程名以":"开头的属于当前应用的私有进程，其他应用的组件不可以和他跑在同一个进程里面。而进程名不以":"开头的进程属于全局进程，其他应用通过ShareUID方式可以和它跑在同一个进程中。两个应用可以通过ShareUID跑在同一个进程并且签名相同，他们可以共享data目录、组件信息、共享内存数据。
  * 多进程通讯的问题：  
1.静态成员和单例模式完全失效。  
2.线程同步机制完全失效。  
3.SharedPreferences的可靠性下降  
4.Application会多次创建  
问题1、2原因是因为进程不同，已经不是同一块内存了；  
问题3是因为SharedPreferences不支持两个进程同事进行读写操作，有一定几率导致数据丢失；  
问题4是当一个组件跑在一个新的进程中，系统会为他创建新的进程同时分配独立的虚拟机，所有这个过程其实就是启动一个应用的过程，，因此相当于系统又把这个应用重新启动了一遍，Application也是新建了。  
实现跨进程通讯有很多方式共享文件、SharedPreferences、基于Binder的Messenger和AIDL、Socket等。

* * *

#### 2.3 IPC基础概念介绍 

##### 2.3.1 Serializable接口 

  * Serializable是Java所提供的一个序列化接口
  * serialVersionUID是用来辅助序列化和反序列化过程的，原则上序列化后的数据中的serialVersionUID要和当前类的serialVersionUID相同才能正常的序列化。
  * 静态成员变量属于类不属于对象，所以不会参加序列化过程；其次用transient关键字标明的成员变量也不参加序列化过程。
  * 重写如下两个方法可以重写系统默认的序列化和反序列化过程
    
    private void writeObject(java.io.ObjectOutputStream out)throws IOException{
    }
    private void readObject(java.io.ObjectInputStream out)throws IOException,ClassNotFoundException{
    }

##### 2.3.2 Parcelable接口 

  * Android中特有的序列化方式，效率相对Serializable更高，占用内存相对也更少，但使用起来稍微麻烦点。
    
    public class Book implements Parcelable {
      public static final Creator<Book> CREATOR = new Creator<Book>() {
          @Override
          public Book createFromParcel(Parcel in) {
              return new Book(in);
          }
          @Override
          public Book[] newArray(int size) {
              return new Book[size];
          }
      };
      public int code;
      public String name;
      public Book(int code, String name) {
          this.code = code;
          this.name = name;
      }
      protected Book(Parcel in) {
          code = in.readInt();
          name = in.readString();
      }
      public int describeContents() {
          return 0;
      }
      @Override
      public void writeToParcel(Parcel dest, int flags) {
          dest.writeInt(code);
          dest.writeString(name);
      }
    }

  * 序列化功能由**writeToParcel**方法来完成，最终通过Parcel中的一系列write方法完成的。反序列化功能由CREATEOR来完成，其内部标明了如何创建序列号对象和诉诸，并通过Parcel的一系列read方法来完成反序列化过程。内容描述功能由describeContents方法来完成，几乎所有情况都返回0，只有当前对象存在文件描述符时，才返回1。

##### 2.3.3 Binder 

  * 从IPC的角度来说，Binder是Android的一种跨进程的通讯方式；Binder也可以理解为是一种虚拟额物理设备，他的设备驱动是/dev/binder；从Android Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager、等等）和ManagerService的桥梁；从Android应用层来说，Binder是客户端与服务端通讯的媒介。在Android开发中，Binder主要用于Service中，包括AIDL和Messenger，其中普通的Service的Binder不涉及进程间通讯；而Messenger的底层其实就是AIDL。
  * 系统会根据AIDL文件生成同名的.java类；首先声明与AIDL文件相对应的几个接口方法，还申明每个接口方法相对应的整形id来做为标识，这几个id在transact过程中标识客户端请求的到底是什么方法。接着会声明一个内部类Stub，这个Stub就是一个Binder类，当客户端和服务端处于同一个进程的时候，方法调用不会走transact过程，处于不同进程时，方法调用会走transact过程，这个逻辑由Stub的内部代理类Proxy来完成。所以核心实现在于它的内部类Stub和Stub的内部代理类Proxy，下面分析其中的方法：  
1.**DESCRIPTOR** Binder的唯一标识，一般用当前Binder的类名表示。  
2.**asInterface(android.os.IBinder obj)** 将服务端的Binder对象转换成客户端所需要的AIDL接口类型的对象；如果客户端和服务端位于相同进程，那么此方法返回的就是服务端Stub对象本身，否则返回系统封装后的Stub.proxy对象。  
3.**asBinder** 用于返回当前的Binder对象  
4.**onTransact** 运行在服务端的Binder线程池中，当客户端发起跨进程通讯时，远程请求会通过系统底层封装交由此方法处理。
    
    public Boolean onTransact(int code,Parcelable data,Parcelable reply,int flags)

服务端通过code确认客户端请求的目标方法是什么，接着从data中取出目标方法所需的参数（如果有），然后执行目标方法。当目标方法执行完后，向reply中写入返回值（如果有）。如果方法返回值为false，那么服务端的请求会失败，利用这个特性我们可以来做**权限验证**。  
5.Proxy#[Method] 代理类中的接口方法。首先创建该方法所需要的输入型参数Parcel对象**_data**和输出型参数Parcel对象**_reply**，然后把参数写入_data中，接着调用transact方法来发起RPC（远程过程调用）请求，同时当前线程挂起；然后服务端的onTransace方法会被调用直到RPC过程返回后，当前线程继续执行，并从_reply中去除RPC过程的返回结果。
  * Binder的两个重要方法**linkToDeath**和**unlinkToDeath**。通过linkToDeath可以给Binder设置一个死亡代理，当Binder死亡时，我们就会收到通知，然后就可以重新发起连接请求。声明一个DeathRecipient对象，DeathRecipient是一个接口，其内部只有一个方法binderDied，实现这个方法后就可以在Binder死亡的时候收到通知了。
    
    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient(){
      @Override
      public void binderDied(){
          if(mBookManager == null){
              return;
          }
          mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
          mBookManager = null;
          // TODO：接下来重新绑定远程Service
      }
    }

在客户端绑定远程服务成功后，给Binder设置死亡代理
    
    mService = IBookManager.Stub.asInterface(binder);
    binder.linkToDeath(mDeathRecipient,0);

  * 当清楚AIDL接口文件的结构和作用后，是可以不通过AIDL而直接实现Binder来进行跨进程通讯的。细节请参考原书以及[随书代码][1]

   [1]: https://github.com/singwhatiwanna/android-art-res/tree/master/Chapter_2/src/com/ryg/chapter_2/manualbinder

* * *

#### 2.4 Android的IPC方式 

  * 使用**Bundle**：由于Binder实现了Parcelable接口，所以可以方便的在不同进程中传输；Activity、Service和Receiver都支持在Intent中传递Bundle数据。
  * 使用**文件共享**：两个进程通过读/写一个文件来交换数据；适合对数据同步要求性不高的场景；并要避免并发写这种场景或者处理好线程同步问题。SharedPreferences是个特例，虽然也是文件的一种，但系统在内存中有一份SharedPreferences文件的缓存，因此在多线程模式下，系统对他的读/写就变得不可靠，高并发读写SharedPreferences有一定几率会丢失数据，因此不建议在多进程通讯时采用SharedPreferences。
  * 使用**Messenger**：Messenger是轻量级的IPC方案，底层实现是AIDL，他对AIDL进行了封装，Messenger 服务端是以串行的方式来处理客户端的请求的，不存在并发执行的情形。
  * 使用**AIDL**：**服务端**首先创建一个Service用来监听客户端的连接请求，然后创建一个AIDL文件，将暴露给客户端的接口在AIDL文件中声明，最后在Service中实现这个AIDL接口即可。**客户端**首先绑定服务端的Service，绑定成功后，将服务端返回的Binder对象转化成AIDL接口所属的类型，调用相对应的AIDL中的方法。  
1.AIDL支持的数据类型：  
基本数据类型；  
String、CharSequence；  
List 只支持ArrayList,里面的元素必须都能被AIDL所支持；  
Map 只支持HashMap，里面的元素（key和value）必须都能被AIDL所支持；Parcelable 所有实现了Parcelable接口的对象；  
AIDL 所有AIDL接口本身也可以在AIDL文件中使用。  
2.自定义的Parcelable对象和AIDL对象必须显示的import进来（即使在同一个包）。  
3.除了基本数据类型，需要用inout表示输入输出型参数。  
4.为了方便AIDL开发，建议把所有和AIDL相关的类和文件都放在同一个包中，好处在于，当客户端是另一个应用的时候，我们可以直接把整个包复制到客户端工程中去。  
5.RemoteCallbackList是系统专门提供用于删除跨进程listener的接口，RemoteCallbackList是泛型，支持管理任意的AIDL接口，因为所有AIDL接口都继承自android.os.IInterface接口。  
6.需注意AIDL客户端发起RPC过程的时候，客户端的线程会挂起，如果是UI线程发起的RPC过程，如果服务端处理事件过长，就会导致ANR。 

  * 使用**ContentProvider**  
ContentProvider是Android专门用于不同应用之间进行数据共享的方式，天生适合跨进程通讯，底层同样采用Binder实现。  
ContentProvider主要以表格的形式来组织数据，可以包含多个表；ContentProvider支持普通文件，甚至可以采用内存中得一个对象来进行数据存储。  
通过ContentProvider的notifyChange方法来通知外界当前ContentProvider中的数据已经发生改变。 

  * 使用**Socket**  
Socket也被称为“套接字”，是网络通讯中得概念，分为流式套接字和用户数据报套接字两种，分别对应网络的传输控制层中得TCP和UDP协议。

以上内容详细的代码示例请参照原书和[随书源码][2]哟。 

   [2]: https://github.com/singwhatiwanna/android-art-res/tree/master/Chapter_2/src/com/ryg/chapter_2/

* * *

#### 2.5 Binder连接池 

  * 当项目越来越庞大后，需要使用到的AIDL接口文件也越来越多，但我们不能有多少个AIDL就添加多少个Service，Service是四大组件之一，是一种系统资源，太多的Service会让我们的App看起来很重量级；我们应该把所有AIDL放在一个Service中去管理。 这时候不同业务模块之间是不能有耦合的，所有实现细节需要单独来开，然后向服务端提供一个queryBInder接口，这个接口根据业务模块的特征来返回Binder对象给它们，不同的业务模块拿到所需的Binder对象给它们，不同的业务模块拿到所需的Binder对象后就可以进行远程方法调用了；由此可见Binder连接池的**主要作用**就是将每个业务模块的Binder请求统一转发到远程Service中去执行。
  * 当新业务模块加入新的AIDL，那么在它实现自己的AIDL接口后，只需要修改BinderPoolImpl中的queryBinder方法，给自己添加一个新的binderCode并返回相对应的Binder对象即可，不需要添加新的Service。作者建议在AIDL开发过程中引入BinderPool机制。 详细源码请参考原书和[随书源码][3]。

   [3]: https://github.com/singwhatiwanna/android-art-res/tree/master/Chapter_2/src/com/ryg/chapter_2/binderpool

* * *

#### 2.6 选用合适的IPC方式 

![][4]  


   [4]: http://upload-images.jianshu.io/upload_images/626583-b6962b1f9a066020.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240

IPC方式的优缺点和适用场景.PNG 

