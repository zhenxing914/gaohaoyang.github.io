---
layout: post
title:  "IO包中的Decorator模式"
categories: "设计模式"
tags: "设计模式 装饰者模式"
author: "songzhx"
date:   2018-07-19 18:12:00
---

JDK为程序员提供了大量的类库，而为了保持类库的可重用性，可扩展性和灵活性，其中使用到了大量的设计模式，本文将介绍JDK的I/O包中使用到的Decorator模式，并运用此模式，实现一个新的输出流类。   　　

**Decorator模式简介**   　　

Decorator模式又名包装器(Wrapper)，它的主要用途在于给一个对象动态的添加一些额外的职责。与生成子类相比，它更具有灵活性。 有时候，我们需要为一个对象而不是整个类添加一些新的功能，比如，给一个文本区添加一个滚动条的功能。我们可以使用继承机制来实现这一功能，但是这种方法不够灵活，我们无法控制文本区加滚动条的方式和时机。而且当文本区需要添加更多的功能时，比如边框等，需要创建新的类，而当需要组合使用这些功能时无疑将会引起类的爆炸。  　　

我们可以使用一种更为灵活的方法，就是把文本区嵌入到滚动条中。而这个滚动条的类就相当于对文本区的一个**装饰**。这个装饰(滚动条)必须与被装饰的组件(文本区)继承自同一个接口，这样，用户就不必关心装饰的实现，因为这对他们来说是透明的。装饰会将用户的请求转发给相应的组件(即调用相关的方法)，并可能在转发的前后做一些额外的动作(如添加滚动条)。通过这种方法，我们可以根据组合对文本区嵌套不同的装饰，从而添加任意多的功能。这种动态的对对象添加功能的方法不会引起类的爆炸，也具有了更多的灵活性。  　　

以上的方法就是**Decorator模式**，它通过给对象添加装饰来动态的添加新的功能。如下是Decorator模式的UML图：

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fd0ly5jtj30av080wep.jpg)

Component为组件和装饰的公共父类，它定义了子类必须实现的方法。  　　ConcreteComponent是一个具体的组件类，可以通过给它添加装饰来增加新的功能。  　　

Decorator是所有装饰的公共父类，它定义了所有装饰必须实现的方法，同时，它还保存了一个对于Component的引用，以便将用户的请求转发给Component，并可能在转发请求前后执行一些附加的动作。  　　

ConcreteDecoratorA和ConcreteDecoratorB是具体的装饰，可以使用它们来装饰具体的Component。



**Java IO包中的Decorator模式**  　　

JDK提供的java.io包中使用了Decorator模式来实现对各种输入输出流的封装。以下将以java.io.OutputStream及其子类为例，讨论一下Decorator模式在IO中的使用。  

首先来看一段用来创建IO流的代码：

以下是代码片段：

```java
try { 
　OutputStream out = new DataOutputStream(new FileOutputStream("test.txt")); 
} catch (FileNotFoundException e) { 
　e.printStackTrace(); 
}
```



这段代码对于使用过JAVA输入输出流的人来说再熟悉不过了，我们使用DataOutputStream封装了一个FileOutputStream。这是一个典型的Decorator模式的使用，FileOutputStream相当于Component，DataOutputStream就是一个Decorator。将代码改成如下，将会更容易理解：

以下是代码片段：

```java
try { 
　OutputStream out = new FileOutputStream("test.txt"); 
　out = new DataOutputStream(out); 
} catch(FileNotFoundException e) { 
　e.printStatckTrace(); 
}
```

由于FileOutputStream和DataOutputStream有公共的父类OutputStream，因此对对象的装饰对于用户来说几乎是透明的。下面就来看看OutputStream及其子类是如何构成Decorator模式的：

![img](https://tva1.sinaimg.cn/large/006y8mN6gy1g6fd0mv0cfj30zq0f8gmo.jpg)

OutputStream是一个抽象类，它是所有输出流的公共父类，其源代码如下：  

以下是代码片段：

```java
public abstract class OutputStream implements Closeable, Flushable { 
public abstract void write(int b) throws IOException; 
... 
} 
```

它定义了write(int b)的抽象方法。这相当于Decorator模式中的Component类。

   ByteArrayOutputStream，FileOutputStream 和 PipedOutputStream 三个类都直接从OutputStream继承，以ByteArrayOutputStream为例： 

以下是代码片段： 

```java
public class ByteArrayOutputStream extends OutputStream { 
protected byte buf[]; 
protected int count; 
public ByteArrayOutputStream() { 
this(32); 
} 
public ByteArrayOutputStream(int size) { 
if (size 〈 0) { 
throw new IllegalArgumentException("Negative initial size: " + size); 
} 
buf = new byte[size]; 
} 
public synchronized void write(int b) { 
int newcount = count + 1; 
if (newcount 〉 buf.length) { 
byte newbuf[] = new byte[Math.max(buf.length 〈〈 1, newcount)]; 
System.arraycopy(buf, 0, newbuf, 0, count); 
buf = newbuf; 
} 
buf[count] = (byte)b; 
count = newcount; 
} 
... 
} 
```

它实现了OutputStream中的write(int b)方法，因此我们可以用来创建输出流的对象，并完成特定格式的输出。它相当于Decorator模式中的ConcreteComponent类。 

  接着来看一下FilterOutputStream，代码如下：   

`以下是代码片段：`

```java
public class FilterOutputStream extends OutputStream { 
protected OutputStream out; 
public FilterOutputStream(OutputStream out) { 
this.out = out; 
} 
public void write(int b) throws IOException { 
out.write(b); 
} 
... 
} 
```

​       同样，它也是从OutputStream继承。但是，它的构造函数很特别，需要传递一个OutputStream的引用给它，并且它将保存对此对象的引用。而如果没有具体的OutputStream对象存在，我们将无法创建FilterOutputStream。由于out既可以是指向FilterOutputStream类型的引用，也可以是指向ByteArrayOutputStream等具体输出流类的引用，因此使用多层嵌套的方式，我们可以为ByteArrayOutputStream添加多种装饰。这个FilterOutputStream类相当于Decorator模式中的Decorator类，它的write(int b)方法只是简单的调用了传入的流的write(int b)方法，而没有做更多的处理，因此它本质上没有对流进行装饰，所以继承它的子类必须覆盖此方法，以达到装饰的目的。  



BufferedOutputStream 和 DataOutputStream是FilterOutputStream的两个子类，它们相当于Decorator模式中的ConcreteDecorator，并对传入的输出流做了不同的装饰。以BufferedOutputStream类为例： 

以下是代码片段： 

```java
public class BufferedOutputStream extends FilterOutputStream {
... 
private void flushBuffer() throws IOException { 
if (count 〉 0) { 
out.write(buf, 0, count); 
count = 0; 
} 
} 
public synchronized void write(int b) throws IOException { 
if (count 〉= buf.length) { 
flushBuffer(); 
} 
buf[count++] = (byte)b; 
} 
... 
}
```

​     这个类提供了一个缓存机制，等到缓存的容量达到一定的字节数时才写入输出流。首先它继承了FilterOutputStream，并且覆盖了父类的write(int b)方法，在调用输出流写出数据前都会检查缓存是否已满，如果未满，则不写。这样就实现了对输出流对象动态的添加新功能的目的。  　　

​    下面，将使用Decorator模式，为IO写一个新的输出流。

**自己写一个新的输出流** 

 　　了解了OutputStream及其子类的结构原理后，我们可以写一个新的输出流，来添加新的功能。这部分中将给出一个新的输出流的例子，它将过滤待输出语句中的空格符号。比如需要输出"java io OutputStream"，则过滤后的输出为"javaioOutputStream"。以下为SkipSpaceOutputStream类的代码：

  `以下是代码片段：`

```java
import java.io.BufferedInputStream; 
import java.io.DataInputStream; 
import java.io.DataOutputStream; 
import java.io.IOException; 
import java.io.InputStream; 
import java.io.OutputStream; 
/** 
* Test the SkipSpaceOutputStream. 
* @author Magic 
* 
*/ 
public class Test { 
　public static void main(String[] args){ 
　　byte[] buffer = new byte[1024]; 

　　/** 
　　* Create input stream from the standard input. 
　　*/ 
　　InputStream in = new BufferedInputStream(new DataInputStream(System.in)); 

　　/** 
　　* write to the standard output. 
　　*/ 
　　OutputStream out = new SkipSpaceOutputStream(new DataOutputStream(System.out)); 

　　try { 
　　　System.out.println("Please input your words: "); 
　　　int n = in.read(buffer,0,buffer.length); 
　　　for(int i=0;i〈n;i++){ 
　　　　out.write(buffer[i]); 
　　　} 
　　} catch (IOException e) { 
　　　e.printStackTrace(); 
　　} 
　} 
}
```



　　执行以上测试程序，将要求用户在console窗口中输入信息，程序将过滤掉信息中的空格，并将最后的结果输出到console窗口。比如：

 `以下是引用片段：`

`Please input your words: `

`a b c d e f `

`abcdef` 　　

**总 结**  

　　在java.io包中，不仅OutputStream用到了Decorator设计模式，InputStream，Reader，Writer等都用到了此模式。而作为一个灵活的，可扩展的类库，JDK中使用了大量的设计模式，比如在Swing包中的MVC模式，RMI中的Proxy模式等等。对于JDK中模式的研究不仅能加深对于模式的理解，而且还有利于更透彻的了解类库的结构和组成。



引用地址：

[Java IO 包中Decorator模式](http://tech.163.com/05/1215/09/250KAFGP0009159F.html)