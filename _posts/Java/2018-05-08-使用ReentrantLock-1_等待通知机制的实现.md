---
layout: post
title:  "使用ReentrantLock-1.等待通知机制的实现"
categories: "Java"
tags: "java 多线程"
author: "songzhx"
date:   2018-05-08 15:33:00
---

MyThreadA 源码

```java
package extthread;

import service.MyService;

public class MyThreadA extends Thread {

	private MyService myService;

	public MyThreadA(MyService myService) {
		super();
		this.myService = myService;
	}

	@Override
	public void run() {
		myService.waitMethod();
	}

}
```



MyService源码

```java
package service;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyService {

	private Lock lock = new ReentrantLock();
	public Condition condition = lock.newCondition();

	public void await() {
		try {
			lock.lock();
			System.out.println(" await时间为：" + System.currentTimeMillis());
			condition.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}

	public void signal() {
		try {
			lock.lock();
			System.out.println("signal时间为：" + System.currentTimeMillis());
			condition.signal();
		} finally {
			lock.unlock();
		}
	}
}
```



Run源码

```java
package test;

import service.MyService;
import extthread.ThreadA;

public class Run {

	public static void main(String[] args) throws InterruptedException {

		MyService service = new MyService();

		ThreadA a = new ThreadA(service);
		a.start();

		Thread.sleep(3000);

		service.signal();

	}

}

```

