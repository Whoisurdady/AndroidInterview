# Android相关

## 1.生命周期

### ActivityA -> ActivityB

A.onPause() -> B.onCreate() ->B.onStart()->B.onResume()->A.onStop()

### ActivityA -> ActivityB(SingleTask) -> ActivityC -> ActivityD -> ActivityB

最后一步D跳转到B，会导致C,D出栈。生命周期如下：

C.onDestroy() -> D.onPause() -> B.onNewIntent() -> B.onStart() -> B.onResume() -> D.onStop() -> D.onDestroy()

## 2.java对象的初始化过程

一个类及其对象初始化的过程
###	一、什么时候需要初始化一个类

首次创建某个对象时 —> Dog dog = new Dog(); 

首次访问某个类的静态方法或者静态字段时 —> Dog.staticFields;

java解释器就会去找类的路径，定位已经编译好的Dog.class文件。

###	二、获得类的资源

然后jvm就会载入Dog.class，生成一个class对象。这个时候如果有静态的方法或者变量，静态初始化动作都会被执行。这个时候要注意啦，静态初始化在程序运行过程中只会在Class对象首次加载的时候运行一次。这些资源都会放在jvm的方法区。

方法区: 

- 又叫静态区，跟堆一样，被所有的线程共享。 
- 方法区中包含的都是在整个程序中永远唯一的元素，包含所有的class和static变量。

###	三、初始化对象—>Dog dog = new Dog()

1.	第一次创建Dog对象先执行上面的一二步 
2.	在堆上为Dog对象分配足够的存储空间，所有属性和方法都被设置成缺省值(数字为0，字符为null，布尔为false，而所有引用被设置成null） 
3.	执行构造函数检查是否有父类，如果有父类会先调用父类的构造函数，这里假设Dog没有父类，执行缺省值字段的赋值即方法的初始化动作。 
4.	执行构造函数。

有父类情况下的初始化
假设Dog extends Animal

执行第一步，找出Dog.class文件，接着在加载过程中发现他有一个基类(通过extends关键字)，于是先执行Animal类的第一二步，加载其静态变量和方法，加载结束之后再加载子类Dog的静态变量和方法。 
如果Animal类还有父类就以此类推，最终的基类叫做根基类。

ps：因为子类的static初始化可能会依赖于父类的静态资源，所以要先加载父类的静态资源。

接着要new Dog对象，先为Dog对象分配存储空间->到Dog的构造函数->创建缺省的属性。这里其构造函数里面的第一行有个隐含的super()，即父类构造函数，所以这时会跳转到父类Animal的构造函数。

java会帮我们完成构造函数的补充，Dog实际隐式的构造函数如下

		Dog() { 
		//创建缺省的属性和方法 
		//调用父类的构造函数super()（可显式写出） 
		//对缺省属性和方法分别进行赋值和初始化 
		}

父类Animal执行构造函数前也是分配存储空间- >到其构造函数->创建缺省的属性->发现挖槽我已经没有父类了，这个时候就给它的缺省的属性赋值和方法的初始化。

接着执行构造函数余下的部分，结束后跳转到子类Dog的构造函数。

子类Dog对缺省属性和方法分别进行赋值和初始化，接着完成构造函数接下来的部分。

*	一、为什么要执行父类Animal的构造方法才继续子类Dog的属性及方法赋值？ 
	
	-因为子类Dog的非静态变量和方法的初始化有可能使用到其父类Animal的属性或方法，所以子类构造缺省的属性和方法之后不应该进行赋值，而要跳转到父类的构造方法完成父类对象的构造之后，才来对自己的属性和方法进行初始化。 
	
	-这也是为什么子类的构造函数显示调用父类构造函数super()时要强制写在第一行的原因，程序需要跳转到父类构造函数完成父类对象的构造后才能执行子类构造函数的余下部分。

*	二、为什么对属性和方法初始化之后再执行构造函数其他的部分？ 
	
	-因为构造函数中的显式部分有可能使用到对象的属性和方法。


	Tips：其实这种初始化过程都是为了保证后面资源初始化用到的东西前面的已经初始化完毕了。很厉害，膜拜java的父亲们。

说了这么多还是来个例子吧。

这里注意main函数也是一个静态资源，执行Dog类的main函数就是调用Dog的静态资源。

		//父类Animal
		class Animal {  
		/*8、执行初始化*/  
		    private int i = 9;  
		    protected int j;  
		
		/*7、调用构造方法，创建缺省属性和方法，完成后发现自己没有父类*/  
		    public Animal() {  
		/*9、执行构造方法剩下的内容，结束后回到子类构造函数中*/  
		        System.out.println("i = " + i + ", j = " + j);  
		        j = 39;  
		     }  
		
		/*2、初始化根基类的静态对象和静态方法*/  
		    private static int x1 = print("static Animal.x1 initialized");  
		    static int print(String s) {  
		        System.out.println(s);  
		        return 47;  
		    }  
		}  
		
		//子类Dog
		public class Dog extends Animal {  
		/*10、初始化缺省的属性和方法*/ 
		    private int k = print("Dog.k initialized");  
		
		/*6、开始创建对象，即分配存储空间->创建缺省的属性和方法。 
		     * 遇到隐式或者显式写出的super()跳转到父类Animal的构造函数。
		     * super()要写在构造函数第一行 */  
		    public Dog() { 
		/*11、初始化结束执行剩下的语句*/
		        System.out.println("k = " + k);  
		        System.out.println("j = " + j);  
		    }  
		
		/*3、初始化子类的静态对象静态方法，当然mian函数也是静态方法*/  
		    private static int x2 = print("static Dog.x2 initialized");
		
		/*1、要执行静态main，首先要加载Dog.class文件，加载过程中发现有父类Animal， 
		     *所以也要加载Animal.class文件，直至找到根基类，这里就是Animal*/       
		    public static void main(String[] args) {  
		
		/*4、前面步骤完成后执行main方法，输出语句*/ 
		      System.out.println("Dog constructor"); 
		/*5、遇到new Dog()，调用Dog对象的构造函数*/  
		        Dog dog = new Dog();   
		/*12、运行main函数余下的部分程序*/            
		        System.out.println("Main Left"); 
		    }  
	}  

输出为：

		static Animal.x1 initialized 
		static Dog.x2 initialized 
		Dog constructor 
		Animal: animal.i = 9, animal.j = 0 
		Dog.k initialized 
		Dog：Dog.k = 47 
		Dog: Animal.j = 39 
		Main Left

##	 3.Binder机制，AIDL

### Binder C/S 架构图

![/img](https://img-blog.csdn.net/20160526102248373)

### Binder如何实现一次数据拷贝？
利用**虚拟内存**。Binder借助内存映射，在内核空间和接收方的用户空间的数据缓存区做了一层内存映射。也就是说，在发送方将数据拷贝到内存空间的时候，**内核空间的这部分地址同时也会被映射到接收方的内存缓存中**，这样子，就少了一次从内和空间拷贝到用户空间
![/img](https://user-gold-cdn.xitu.io/2018/8/11/16528ee0ccf0a781?imageslim)
### 什么是AIDL

[AIDL](https://developer.android.com/guide/components/aidl)

### View的绘制过程

[绘制流程](http://blog.csdn.net/qinjuning/article/details/7110211/)

![img](https://upload-images.jianshu.io/upload_images/5064108-6894ba77dc36744e.png?imageMogr2/auto-orient/)

### Linearlayout 里的weight

weight是指对父view中的额外空间（注意这个额外空间）按比例分配给每个view
,尤其是matchParrent的时候，如果有多个matchParent的子view，额外空间是负数（父width-n*子width）

[Linearlayout.weight](http://blog.csdn.net/goodlixueyong/article/details/50004837)

### Hanlder, Looper, MessageQueue

读源码吧，主要关注delay如何实现，Looper如何被唤醒

### Launch Mode
1.	**standard**:	不管有没有已存在的实例，都生成新的实例
2. **singleTop**: 如果发现有对应的Activity实例正位于栈顶，则重复利用，不再生成新的实例。否则同**standard**
3. **singleTask**: 如果发现有对应的Activity实例，则使此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象，显示到幕前
4. **singleInstance**: 它会启用一个新的栈结构，将Acitvity放置于这个新的栈结构中，并保证不再有其他Activity实例进入

### SurfaceVIew 和 TextureView
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

### RecyclerView相关

1.	如何重写layoutManager实现复杂的布局(如瀑布流)
2. Recycler工作原理(holder缓存，复用机制)
3. ItemAnimaton

### JVM & GC
1. 引用计数
2. 根搜算法
3. CounterMarkSwap(CMS)
4. 内存分代
5. 堆栈

### 第三方源码

1.	okhttp
2. glide
3. eventbus

### MVP & MVC & MVVM

### android 打包流程
[打包流程](http://blog.csdn.net/huachao1001/article/details/51504469)

![img](https://upload-images.jianshu.io/upload_images/3385286-48c785a0682c408b.png?imageMogr2/auto-orient/)