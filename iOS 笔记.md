## GCD 死锁

- GCD 死锁的充分条件是:“向当前队列重复同步提交 block”。从原理来看，死锁的原因是提交的 block 阻塞了队列，而队列阻塞后永远无法执行完 `dispatch_sync()`，可见这里完全和代码所在的线程无关。

  （参考：https://bestswifter.com/zhu-xian-cheng-zhong-ye-bu-jue-dui-an-quan-de-ui-cao-zuo/，dispatch_sync API）

- 在主线程中向一个串行队列同步的派发 block (异步派发则不会)，根据上文选择线程的原则，block 将在主线程中执行，但同样不会导致死锁:

```objc
dispatch_queue_t queue = dispatch_queue_create("com.kt.deadlock", nil);  
dispatch_sync(queue, ^{  
    NSLog(@"current thread = %@", [NSThread currentThread]);
});
// 输出结果:
// current thread = <NSThread: 0x7fa7fb403e90>{number = 1, name = main} 
```

原因是：这都是两个队列，所以不要求任务A执行完再执行任务B,由于是同步，所以是先执行任务A的一部分，再去执行任务B的一部分，最后再去执行任务A,只不过这个过程都是主线程执行而已。

https://segmentfault.com/q/1010000005904095



## iOS几种延迟执行方法的比较

**performSelector** 和 **NSTimer** 需要在 runloop 中执行

https://juejin.im/post/5a31d4d751882554bd510ff1



## OC 中 `load` 方法和 `initialize` 方法的异同

- `load`方法在这个文件被程序装载时调用。只要是在Compile Sources中出现的文件总是会被装载，这与这个类是否被用到无关，因此`load`方法总是在`main`函数之前调用。
- `initialize` 这个方法在第一次给某个类发送消息时调用（比如实例化一个对象），并且只会调用一次。`initialize`方法实际上是一种惰性调用，也就是说如果一个类一直没被用到，那它的`initialize`方法也不会被调用，这一点有利于节约资源。

https://bestswifter.com/bat-interview/



## ContentInset

```
contentInset：这个属性能够在UIScrollView的4周增加额外的滚动区域，一般用来避免scrollView的内容被其他控件挡住
```



## 砸壳

为了砸壳，我们需要使用到 **dumpdecrypted**，这个工具已经开源并且托管在了 GitHub 上面。

1. 使用 ssh 连上你的 iOS 系统

2. 连上之后，使用 `ps` 配合 `grep` 命令来找到微信的可执行文件

3. 使用 **Cycript** 找到目标应用的 Documents 目录路径。

4. 将 **dumpdecrypted.dylib** 拷到 Documents 目录下：

5. 砸壳

   ```
   $ cd /var/mobile/Containers/Data/Application/13727BC5-F7AA-4ABE-8527-CEDDA5A1DADD/Documents
   # 后面的路径即为一开始使用 ps 命令找到的目标应用可执行文件的路径 
   $ DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/944128A6-C840-434C-AAE6-AE9A5128BE5B/WeChat.app/WeChat
   ```

    完成后会在当前目录生成一个 WeChat.decrypted 文件，这就是砸壳后的文件。之后就是将它拷贝到 OS X 用 class-dump 来导出头文件啦。

http://www.swiftyper.com/2016/05/02/iOS-reverse-step-by-step-part-1-class-dump/



## 谈谈**instancetype和id的异同** 

1. **相同点** 
 都可以作为方法的返回类型

2. **不同点**
①instancetype可以返回和方法所在类相同类型的对象，id只能返回未知类型的对象；
②instancetype只能作为返回值，不能像id那样作为参数



## 谈谈load和initialize的区别

![load_vs_initialize](/Users/kira/Desktop/iOS_Notes/load_vs_initialize.png)

## 在一个app中间有一个button，在你手触摸屏幕点击后，到这个button收到点击事件，中间发生了什么
```
响应链大概有以下几个步骤

1. 设备将touch到的UITouch和UIEvent对象打包, 放到当前活动的Application的事件队列中
2. 单例的UIApplication会从事件队列中取出触摸事件并传递给单例UIWindow
3. UIWindow使用hitTest:withEvent:方法查找touch操作的所在的视图view

RunLoop这边我大概讲一下

1. 主线程的RunLoop被唤醒
2. 通知Observer，处理Timer和Source 0
3. Springboard接受touch event之后转给App进程中
4. RunLoop处理Source 1，Source1 就会触发回调，并调用_UIApplicationHandleEventQueue() 进行应用内部的分发。
5. RunLoop处理完毕进入睡眠，此前会释放旧的autorelease pool并新建一个autorelease pool

深挖请去[深入理解RunLoop](https://link.jianshu.com?t=http://blog.ibireme.com/2015/05/18/runloop/)

UIResponder是UIView的父类，UIView是UIControl的父类。
```

## 静态库，动态库与 Framework

> - https://segmentfault.com/a/1190000004920754

## Objective-C 内存管理

> - https://segmentfault.com/a/1190000004943276