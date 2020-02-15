---
layout: post
title:  "使用ReentrantLock-3.公平锁和非公平锁"
categories: "Java"
tags: "java 多线程 lock"
author: "songzhx"
date:   2018-05-08 17:33:00
---

**公平锁和非公平锁：**

> 公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先的FIFO先进先出的顺序。
>
> 而非公平锁就是一种获取锁的抢占机制，即随机获的锁的，和公平锁不一样的就是先来的不一定能先得到锁，这个方式可能造成某些线程一直拿不到锁，结果也就是不公平的。

```java
package service;

import java.util.concurrent.locks.ReentrantLock;

public class Service {

	private ReentrantLock lock;

	public Service(boolean isFair) {
		super();
		lock = new ReentrantLock(isFair);
	}

	public void serviceMethod() {
		try {
			lock.lock();
			System.out.println("ThreadName=" + Thread.currentThread().getName()
					+ "获得锁定");
		} finally {
			lock.unlock();
		}
	}

}
```

```java
package test.run;

import service.Service;

public class RunFair {

	public static void main(String[] args) throws InterruptedException {
		final Service service = new Service(true);

		Runnable runnable = new Runnable() {
			@Override
			public void run() {
				System.out.println("★线程" + Thread.currentThread().getName()
						+ "运行了");
				service.serviceMethod();
			}
		};

		Thread[] threadArray = new Thread[10];
		for (int i = 0; i < 10; i++) {
			threadArray[i] = new Thread(runnable);
		}
		for (int i = 0; i < 10; i++) {
			threadArray[i].start();
		}

	}
}

```

```java
package test.run;

import service.Service;

public class RunNotFair {

	public static void main(String[] args) throws InterruptedException {
		final Service service = new Service(false);

		Runnable runnable = new Runnable() {
			@Override
			public void run() {
				System.out.println("★线程" + Thread.currentThread().getName()
						+ "运行了");
				service.serviceMethod();
			}
		};

		Thread[] threadArray = new Thread[10];
		for (int i = 0; i < 10; i++) {
			threadArray[i] = new Thread(runnable);
		}
		for (int i = 0; i < 10; i++) {
			threadArray[i].start();
		}

	}
}
```

公平锁运行的结果：

<img src="/Users/song/Library/Application Support/typora-user-images/image-20200113161657454.png" alt="image-20200113161657454" style="zoom:50%;" />

打印的结果基本是呈有序的状态，这就是公平锁的特点。



非公平锁的结果：

<img src="/Users/song/Library/Application Support/typora-user-images/image-20200113161722133.png" alt="image-20200113161722133" style="zoom:50%;" />

非公平锁的运行结果基本上是乱序的，说明先start()启动的线程并不代表先获得锁。