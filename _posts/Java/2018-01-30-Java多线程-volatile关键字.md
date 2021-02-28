---
layout: post
title:  "volatile关键字"
categories: "Java"
tags: "java  volatile关键字"
author: "songzhx"
date:   2018-07-12 13:56

---

>关键字volatile的作用是强制从**公共堆栈**中取得变量的值，而不是从**线程私有数据栈**中取的变量的值。

## 1. 解决死循环问题

```java
package extthread;

public class RunThread extends Thread {

    private boolean isRunning = true;

	public boolean isRunning() {
		return isRunning;
	}

	public void setRunning(boolean isRunning) {
		this.isRunning = isRunning;
	}

	@Override
	public void run() {
		System.out.println("进入run了");
		while (isRunning == true) {
		}
		System.out.println("线程被停止了！");
	}

}

```

````java
package test;

import extthread.RunThread;

public class Run {
	public static void main(String[] args) {
		try {
			RunThread thread = new RunThread();
			thread.start();
			Thread.sleep(1000);
			thread.setRunning(false);
			System.out.println("已经赋值为false");
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

````

当JVM的运行参数为-server，运行会出现死循环的效果。

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcrqmm2aj315w0b4abj.jpg" alt="img" style="zoom: 50%;" />

**JVM设置成-server模式时为了线程运行的效率，线程一直在私有堆栈中取得isRunning的值是true**。而代码thread.setRunning(false)虽然被执行，更新的却是公共堆栈中的isRunning变量值false,所以一直就是死循环的状态。内存结构如图。

这个问题是因为私有堆栈中值和公共堆栈中的值不同步造成的。解决这个问题就要使用volatile关键字，它强制性从公有堆栈中取值。

修改后代码如：

```java
package extthread;

public class RunThread extends Thread {

	volatile private boolean isRunning = true;

	public boolean isRunning() {
		return isRunning;
	}

	public void setRunning(boolean isRunning) {
		this.isRunning = isRunning;
	}

	@Override
	public void run() {
		System.out.println("进入run了");
		while (isRunning == true) {
		}
		System.out.println("线程被停止了！");
	}

}

```

volatile最致命的缺点就是不支持原子性。

synchronized和volatile比较：

1. 关键字volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好，并且volatile只能修饰于变量，而synchronized可以修饰方法，以及代码块。

2. **多线程访问volatile不会发生阻塞，而synchronized会出现阻塞**

3. **volatile能保证数据的可见性，但不能保证原子性，synchronized可以保证原子性**，也可以间接保证可见性，因为它会将私有内存和公有内存中的数据做同步。

4. volatile解决的事变量在多个线程之间的可见性，而synchronized关键字解决的是多个线程之间**访问资源的同步性**。

   

   **线程安全包含原子性和可见性两个方面，java的同步机制都是围绕这个方面来确保线程安全的。**



## 2. volatile非原子的特性

```java
package extthread;

public class MyThread extends Thread {
	volatile public static int count;

	private static void addCount() {
		for (int i = 0; i < 100; i++) {
			count++;
		}
		System.out.println("count=" + count);
	}

	@Override
	public void run() {
		addCount();
	}

}

```

```java
package test.run;

import extthread.MyThread;

public class Run {
	public static void main(String[] args) {
		MyThread[] mythreadArray = new MyThread[100];
		for (int i = 0; i < 100; i++) {
			mythreadArray[i] = new MyThread();
		}

		for (int i = 0; i < 100; i++) {
			mythreadArray[i].start();
		}

	}

}
```

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcrr4txgj30ho0g8dgu.jpg" alt="img" style="zoom:50%;" />

修改实例变量中的数据，比如i++的操作不是一个原子操作，也就是非线程安全的。表达式i++的操作步骤分解成：

1. 从内存中取出i的值
2. 计算i的值
3. 将i的值写到内存中

假如在第二步计算值的时候，另外一个线程也修改了i的值，那么这个时候就会出现脏数据。

变量在内存中工作的过程如图所示：

1. read和load阶段 ： 从主存复制变量到当前线程工作内存

2. use和assign阶段：执行代码，改变共享变量值

3. store和write阶段：用工作内存数据刷新主存对应变量的值

   

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcrro24nj31as0rwtbh.jpg" alt="img" style="zoom: 50%;" />

在多线程环境中，use和assign是多次出现的，但这一操作并不是原子性，也就是在read和load之后，如果主内存count变量发生修改之后，线程工作内存中的值由于已经加载，不会产生对应的变化，也就是私有内存和公共内存中变量不同步，所以计算出来的结果会和预期不一样，就是出现非线程安全问题。

进行改进：

```java
package extthread;

public class MyThread extends Thread {
	public static int count;

	synchronized private static void addCount() {
		for (int i = 0; i < 100; i++) {
			count++;
		}
		System.out.println("count=" + count);
	}

	@Override
	public void run() {
		addCount();
	}

}

```



## 3. 使用原子类进行i++操作

### 3.1  AtomicInteger原子类 i++操作

除了在i++操作时使用synchronized关键字实现同步外，还可以使用AtomicInteger原子类进行实现。

```java
package extthread;

import java.util.concurrent.atomic.AtomicInteger;

public class AddCountThread extends Thread {
	private AtomicInteger count = new AtomicInteger(0);

	@Override
	public void run() {
		for (int i = 0; i < 10000; i++) {
			System.out.println(count.incrementAndGet());
		}
	}
}


package test;

import extthread.AddCountThread;

public class Run {

	public static void main(String[] args) {
		AddCountThread countService = new AddCountThread();

		Thread t1 = new Thread(countService);
		t1.start();

		Thread t2 = new Thread(countService);
		t2.start();

		Thread t3 = new Thread(countService);
		t3.start();

		Thread t4 = new Thread(countService);
		t4.start();

		Thread t5 = new Thread(countService);
		t5.start();

	}

}

```

### 3.2 原子类也并不安全

原子类在具有逻辑性的情况下输出结果也具有随机性。可能出现打印顺序出错问题。

```java
package service;

import java.util.concurrent.atomic.AtomicLong;

public class MyService {

	public static AtomicLong aiRef = new AtomicLong();

	public void addNum() {
		System.out.println(Thread.currentThread().getName() + "加了100之后的值是:"
				+ aiRef.addAndGet(100));
		aiRef.addAndGet(1);
	}

}



package extthread;

import service.MyService;

public class MyThread extends Thread {
	private MyService mySerivce;

	public MyThread(MyService mySerivce) {
		super();
		this.mySerivce = mySerivce;
	}

	@Override
	public void run() {
		mySerivce.addNum();
	}

}


package test.run;

import service.MyService;
import extthread.MyThread;

public class Run {

	public static void main(String[] args) {
		try {
			MyService service = new MyService();

			MyThread[] array = new MyThread[5];
			for (int i = 0; i < array.length; i++) {
				array[i] = new MyThread(service);
			}
			for (int i = 0; i < array.length; i++) {
				array[i].start();
			}
			Thread.sleep(1000);
			System.out.println(service.aiRef.get());
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

```

输出的结果如下所示：

<img src="https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcrsg32xj30fa08ejru.jpg" alt="image-20190131092439358" style="zoom: 67%;" />

打印顺序出错了，应该是每次加100再加1。出现这样的情况是因为addAndGet()方法是原子的，但是方法和方法之间的调用却不是原子的。解决这样的问题必须要用同步。

```java
package service;

import java.util.concurrent.atomic.AtomicLong;

public class MyService {

	public static AtomicLong aiRef = new AtomicLong();

	synchronized public void addNum() {
		System.out.println(Thread.currentThread().getName() + "加了100之后的值是:"
				+ aiRef.addAndGet(100));
		aiRef.addAndGet(1);
	}

}
```

## 4. Volatitle保证指令不会重排

### 1. 指令重排



#### 例1.A线程指令重排导致B线程出错
对于在同一个线程内，这样的改变是不会对逻辑产生影响的，但是在多线程的情况下指令重排序会带来问题。看下面这个情景:

在线程A中:
```java
context = loadContext();

inited = true;
```


在线程B中:
```java
while(!inited ){ //根据线程A中对inited变量的修改决定是否使用context变量

   sleep(100);

}

doSomethingwithconfig(context);

```

假设线程A中发生了指令重排序:
```java
inited = true;

context = loadContext();
```


那么B中很可能就会拿到一个尚未初始化或尚未初始化完成的context,从而引发程序错误。



#### 例2.指令重排导致单例模式失效

我们都知道一个经典的懒加载方式的双重判断单例模式：

```java
public class Singleton {

  private static Singleton instance = null;

  private Singleton() { }

  public static Singleton getInstance() {

     if(instance == null) {
    
        synchronzied(Singleton.class) {
    
           if(instance == null) {
    
               instance = new Singleton();  //非原子操作
    
           }
    
        }
    
     }
    
     return instance;

   }

}
```



看似简单的一段赋值语句：instance= new Singleton()，但是很不幸它并不是一个原子操作，其实际上可以抽象为下面几条JVM指令：

```java
memory =allocate();    //1：分配对象的内存空间 

ctorInstance(memory);  //2：初始化对象 

instance =memory;     //3：设置instance指向刚分配的内存地址 
```

 上面操作2依赖于操作1，但是操作3并不依赖于操作2，所以JVM是可以针对它们进行指令的优化重排序的，经过重排序后如下：

```java
memory =allocate();    //1：分配对象的内存空间 

instance =memory;     //3：instance指向刚分配的内存地址，此时对象还未初始化

ctorInstance(memory);  //2：初始化对象
```

可以看到指令重排之后，instance指向分配好的内存放在了前面，而这段内存的初始化被排在了后面。

在线程A执行这段赋值语句，在初始化分配对象之前就已经将其赋值给instance引用，恰好另一个线程进入方法判断instance引用不为null，然后就将其返回使用，导致出错。



### 2. 解决重排问题

**解决方案：**

例子1中的inited和例子2中的instance以关键字volatile修饰之后，就会阻止JVM对其相关代码进行指令重排，这样就能够按照既定的顺序指执行。

volatile关键字通过提供“内存屏障”的方式来防止指令被重排序，为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

大多数的处理器都支持内存屏障的指令。

对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能，为此，Java内存模型采取保守策略。下面是基于保守策略的JMM内存屏障插入策略：

```shell
## 写操作
在每个volatile写操作的前面插入一个StoreStore屏障。

在每个volatile写操作的后面插入一个StoreLoad屏障。

## 读操作
在每个volatile读操作的后面插入一个LoadLoad屏障。

在每个volatile读操作的后面插入一个LoadStore屏障。
```



指令重排序： https://blog.csdn.net/jiyiqinlovexx/article/details/50989328

