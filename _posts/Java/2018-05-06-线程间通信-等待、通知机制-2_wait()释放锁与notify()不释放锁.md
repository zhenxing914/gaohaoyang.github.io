---
layout: post
title:  "线程间通信-等待/通知机制-2.wait()释放锁与notify()不释放锁"
categories: "Java"
tags: "java 多线程"
author: "songzhx"
date:   2018-05-07 16:02:00
---

当方法wait()被执行后，锁被自动释放，但执行完notify()方法，锁却不自动释放。

## 1. 验证wait() 锁自动释放

```java
package extthread;

import service.Service;

public class ThreadA extends Thread {

	private Object lock;

	public ThreadA(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {
		Service service = new Service();
		service.testMethod(lock);
	}

}


```

```java
package extthread;

import service.Service;

public class ThreadB extends Thread {
	private Object lock;

	public ThreadB(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {
		Service service = new Service();
		service.testMethod(lock);
	}

}

```

```java
package service;

public class Service {

	public void testMethod(Object lock) {
		try {
			synchronized (lock) {
				System.out.println("begin wait()");
				lock.wait();
				System.out.println("  end wait()");
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}

```

```java
package test;

import extthread.ThreadA;
import extthread.ThreadB;

public class Test {

	public static void main(String[] args) {

		Object lock = new Object();

		ThreadA a = new ThreadA(lock);
		a.start();

		ThreadB b = new ThreadB(lock);
		b.start();

	}

}

```

执行结果：

![image-20190322161122670](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcrudoisj30hu07ugln.jpg)



## 2. 验证notify()锁不自动释放

   ```java
   
   package extthread;
   
   import service.Service;
   
   public class NotifyThread extends Thread {
      private Object lock;

      public NotifyThread(Object lock) {
        super();
        this.lock = lock;
      }

      @Override
      public void run() {
        Service service = new Service();
        service.synNotifyMethod(lock);
      }
 
   }
   
   ```

   

   

```java
package extthread;

import service.Service;

public class synNotifyMethodThread extends Thread {
	private Object lock;

	public synNotifyMethodThread(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {
		Service service = new Service();
		service.synNotifyMethod(lock);
	}

}

```

```java
package extthread;

import service.Service;

public class ThreadA extends Thread {
	private Object lock;

	public ThreadA(Object lock) {
		super();
		this.lock = lock;
	}

	@Override
	public void run() {
		Service service = new Service();
		service.testMethod(lock);
	}

}

```

```java
package service;

public class Service {

	public void testMethod(Object lock) {
		try {
			synchronized (lock) {
				System.out.println("begin wait() ThreadName="
						+ Thread.currentThread().getName());
				lock.wait();
				System.out.println("  end wait() ThreadName="
						+ Thread.currentThread().getName());
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public void synNotifyMethod(Object lock) {
		try {
			synchronized (lock) {
				System.out.println("begin notify() ThreadName="
						+ Thread.currentThread().getName() + " time="
						+ System.currentTimeMillis());
				lock.notify();
				Thread.sleep(5000);
				System.out.println("  end notify() ThreadName="
						+ Thread.currentThread().getName() + " time="
						+ System.currentTimeMillis());
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}

```

```java
package test;

import extthread.NotifyThread;
import extthread.ThreadA;
import extthread.synNotifyMethodThread;

public class Test {

	public static void main(String[] args) throws InterruptedException {

		Object lock = new Object();

		ThreadA a = new ThreadA(lock);
		a.start();

		NotifyThread notifyThread = new NotifyThread(lock);
		notifyThread.start();

		synNotifyMethodThread c = new synNotifyMethodThread(lock);
		c.start();

	}

}

```

执行结果：

![image-20190322161451029](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fcrvsieej30jm07ujs2.jpg)

此实验说明，必须执行完notify()方法所在的同步synchronized代码块后才释放锁。