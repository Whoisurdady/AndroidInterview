# Android相关

## 1.生命周期

### ActivityA -> ActivityB

A.onPause() -> B.onCreate() ->B.onStart()->B.onResume()->A.onStop()

### ActivityA -> ActivityB(SingleTask) -> ActivityC -> ActivityD -> ActivityB

最后一步D跳转到B，会导致C,D出栈。生命周期如下：

C.onDestroy() -> D.onPause() -> B.onNewIntent() -> B.onStart() -> B.onResume() -> D.onStop() -> D.onDestroy()


##	 2.Binder机制，AIDL

### Binder C/S 架构图

![/img](https://img-blog.csdn.net/20160526102248373)

### Binder如何实现一次数据拷贝？
利用**虚拟内存**。Binder借助内存映射，在内核空间和接收方的用户空间的数据缓存区做了一层内存映射。也就是说，在发送方将数据拷贝到内存空间的时候，**内核空间的这部分地址同时也会被映射到接收方的内存缓存中**，这样子，就少了一次从内和空间拷贝到用户空间
![/img](https://user-gold-cdn.xitu.io/2018/8/11/16528ee0ccf0a781?imageslim)
### 什么是AIDL

[AIDL](https://developer.android.com/guide/components/aidl)

## 3.View的绘制过程

[绘制流程](http://blog.csdn.net/qinjuning/article/details/7110211/)

![img](https://upload-images.jianshu.io/upload_images/5064108-6894ba77dc36744e.png?imageMogr2/auto-orient/)

## 4.Linearlayout 里的weight

weight是指对父view中的额外空间（注意这个额外空间）按比例分配给每个view
,尤其是matchParrent的时候，如果有多个matchParent的子view，额外空间是负数（父width-n*子width）

[Linearlayout.weight](http://blog.csdn.net/goodlixueyong/article/details/50004837)

## 5.Hanlder, Looper, MessageQueue

读源码吧，主要关注delay如何实现，Looper如何被唤醒

## 6.Launch Mode
1.	**standard**:	不管有没有已存在的实例，都生成新的实例
2. **singleTop**: 如果发现有对应的Activity实例正位于栈顶，则重复利用，不再生成新的实例。否则同**standard**
3. **singleTask**: 如果发现有对应的Activity实例，则使此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象，显示到幕前
4. **singleInstance**: 它会启用一个新的栈结构，将Acitvity放置于这个新的栈结构中，并保证不再有其他Activity实例进入

## 7.SurfaceVIew 和 TextureView
[SurfaceView和TextureView详细对比](https://www.jianshu.com/p/b9a1e66e95ea)

性能对比
![img](https://upload-images.jianshu.io/upload_images/11368780-d5bbf663aeca1dba?imageMogr2/auto-orient/)

从性能和安全性角度出发，使用播放器优先选SurfaceView。

1.	在android 7.0上系统surfaceview的性能比TextureView更有优势，支持对象的内容位置和包含的应用内容同步更新，平移、缩放不会产生黑边。 在7.0以下系统如果使用场景有动画效果，可以选择性使用TextureView
2. 由于失效(invalidation)和缓冲的特性，TextureView增加了额外1~3帧的延迟显示画面更新
3. TextureView总是使用GL合成，而SurfaceView可以使用硬件overlay后端，可以占用更少的内存带宽，消耗更少的能量
4. TextureView的内部缓冲队列导致比SurfaceView使用更多的内存
5. SurfaceView： 内部自己持有surface，surface 创建、销毁、大小改变时系统来处理的，通过surfaceHolder 的callback回调通知。当画布创建好时，可以将surface绑定到MediaPlayer中。SurfaceView如果为用户可见的时候，创建SurfaceView的SurfaceHolder用于显示视频流解析的帧图片，如果发现SurfaceView变为用户不可见的时候，则立即销毁SurfaceView的SurfaceHolder，以达到节约系统资源的目的

**结论**: 有动画或者位移，缩放等的场景下用TextureView, 不动的时候用SurfaceView

## 8.RecyclerView相关

1.	如何重写layoutManager实现复杂的布局(如瀑布流)
2. Recycler工作原理(holder缓存，复用机制)
3. ItemAnimaton



## 9.第三方源码

1.	okhttp
2. glide
3. eventbus

## 10.MVP & MVC & MVVM

## 11.android 打包流程
[打包流程](http://blog.csdn.net/huachao1001/article/details/51504469)

![img](https://upload-images.jianshu.io/upload_images/3385286-48c785a0682c408b.png?imageMogr2/auto-orient/)

## 12.android 启动流程
[启动流程](https://www.jianshu.com/p/a5532ecc8377)

![img](https://upload-images.jianshu.io/upload_images/851999-a9c2c456c9f91596.jpg?imageMogr2/auto-orient/)