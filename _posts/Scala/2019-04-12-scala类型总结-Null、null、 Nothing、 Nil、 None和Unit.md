---
layout: post
title:  "scala类型总结-Null、null、 Nothing、 Nil、 None和Unit"
categories: "scala"
tags: "scala"
author: "songzhx"
date:   2019-04-12 10:15:00 
---



```
**原文：**
http://oldfashionedsoftware.com/2008/08/20/a-post-about-nothing/
```



```
您听到的有关Scala语言的主要抱怨之一是与Java相比，它太复杂了。一般的开发者永远无法对类型系统，函数式编程语言等有足够的理解。这就是论点。为了支持这个位置，你会经常听到它指出Scala包含了一些虚无的概念（Null，null，Nil，Nothing，None和Unit），并且你必须知道在不同情况下使用哪一个。我不止一次读过这样的论点。
这不是那么糟糕。是的，这些都是Scala的一部分，是的，你必须在正确的情况下使用正确的。但情况如此千差万别，一旦你知道这些事情是什么意思，就不难了解了。
```

## 1. Null and null

```
首先，我们来处理Null和null。空（Null）是一个trait，它（如果你不熟悉trait）有点像Java中的抽象类。确实存在一个空的实例，那是空的。不那么难 字面null与Java中的相同。它是不引用任何对象的引用的值。因此，如果您编写一个采用Null类型参数的方法，则只能传递两个值：null本身或Null类型的引用。
注意：
```



```scala

scala> def tryit(thing: Null): Unit = { println("That worked!"); }
tryit: (Null)Unit

scala> tryit("hey")
<console>:6: error: type mismatch;
 found   : java.lang.String("hey")
 required: Null
       tryit("hey")
             ^

scala> val someRef: String = null
someRef: String = null

scala> tryit(someRef)
<console>:7: error: type mismatch;
 found   : String
 required: Null
       tryit(someRef)
             ^

scala> tryit(null)
That worked!

scala> val nullRef: Null = null
nullRef: Null = null

scala> tryit(nullRef)
That worked!

```

## 2. Nil

```scala
Nil是一个容易的。Nil是扩展List [Nothing]的对象（下面我们将讨论Nothing）。这是一个空的列表。以下是使用Nil的一些示例代码：

scala> Nil
res4: Nil.type = List()

scala> Nil.length
res5: Int = 0

scala> Nil + "ABC"
res6: List[java.lang.String] = List(ABC)

scala> Nil + Nil
res7: List[object Nil] = List(List())
```

能看到？

它基本上是一个不断封装任何东西的空列表。

这是零长度。

它根本不代表“虚无”。

这是一个名单，一个List。

没有内容。

## 3. Nothing



如果其中任何一个有点难以得到，那就什么都不是。Nothing是另一个trait。它继承了Any类。Any是整个Scala类型系统的**超类**。Any可以引用对象类型以及诸如普通的旧整数或双精度的值。没有Nothing的例子，但是（这里是棘手的一点）Nothing是**一切类的**子类  。Nothing是List的子类，它是String的子类，它是Int的子类型，它是YourOwnCustomClass的子类。

还记得Nil吗？这是一个List [Nothing]，它是空的。由于Nothing是一切类的子类，所以Nil可以被用作一个空的字符串列表，一个空的Ints列表，一个空的Any列表。所以没有什么是有用的定义集合或其他类采取类型参数的基本情况。这里是一个scala会话的片段：

```scala
scala> val emptyStringList: List[String] = List[Nothing]()
emptyStringList: List[String] = List()

scala> val emptyIntList: List[Int] = List[Nothing]()
emptyIntList: List[Int] = List()

scala> val emptyStringList: List[String] = List[Nothing]("abc")
<console>:4: error: type mismatch;
 found   : java.lang.String("abc")
 required: Nothing
       val emptyStringList: List[String] = List[Nothing]("abc")
```

在第1行，我们将一个List [Nothing]分配给对List [String]的引用。Nothing是字符串，所以它正常工作。在第4行，我们将一个List [Nothing]赋值给List [Int]的引用。Nothing也是一个Int，所以这也是一样的。Nothing是一切类的子类。但是这两个List [Nothing]实例都不包含成员。当我们尝试创建一个List [Nothing]包含一个String并将该List分配给一个List [String]引用时会发生什么？它失败了，因为虽然Nothing是所有东西的子类，但它不是任何东西的**超类**，也 **没有** Nothing的实例，包括字符串“abc”。所以Nothing的任何集合都必须是空的。

Nothing的另一个用途是作为***永不***  返回的方法的返回类型  。如果你仔细想想，这是有道理的。如果方法的返回类型是Nothing，并且绝对不存在Nothing的实例，那么这样的方法不能返回。

​	代表非正常退出。比如抛异常。这个也很make sense，异常嘛，肯定啥都返回不了，类型只有是Nothing了。

```scala
scala> def e = throw new Exception("s")
e: Nothing
```

## 4. None

当你用Java写一个函数，遇到你没有什么实用价值的情况时，你会怎么做？有几种方法来处理它。你可以返回null，但是这会导致问题。如果调用者不希望得到null，那么当他尝试使用它时可能会面临**NullPointerException**，否则调用者必须检查null。一些函数肯定不会返回null，但有些函数可能会。作为一个调用者，你不知道。

有一种方法可以在函数签名中声明，你可能无法返回一个很好的值，throws关键字。但是try / catch块有一个相关的成本，而且你通常希望保留异常情况下的使用，而不仅仅是为了表示一个普通的无结果的情况。

Scala有这个问题的内置解决方案。例如，如果你想返回一个字符串，但你知道你可能无法返回一个合理的值，你可以返回一个Option[String]。这是一个简单的例子。

```scala
scala> def getAStringMaybe(num: Int): Option[String] = {
     |   if ( num >= 0 ) Some("A positive number!")
     |   else None // A number less than 0?  Impossible!
     | }

getAStringMaybe: (Int)Option[String]

scala> def printResult(num: Int) = {
     |   getAStringMaybe(num) match {
     |     case Some(str) => println(str)
     |     case None => println("No string!")
     |   }
     | }
printResult: (Int)Unit

scala> printResult(100)
A positive number!

scala> printResult(-50)
No string!
```

getAStringMaybe方法返回Option [String]。Option是一个具有两个子类的抽象类，类Some和对象None。这是实例化Option的唯一两种方法。所以getAStringMaybe可以返回一个Some [String]或None。Some和None是大小写类，所以你可以使用方便的match / case结构来处理结果。没有一个对象表示没有方法的结果。

Option [T]返回类型的目的是告诉调用者该方法可能以Some [T]的形式返回T，或者返回None来表示没有结果。通过这种方式，调用者可以知道他什么时候做，不需要检查一个好的返回值。

另一方面，只是因为一个方法被声明为返回一些非Option类型并不意味着它不能返回null。而且，声明为返回Option的方法实际上可以返回null。所以这项技术并不完美。

这是一个巧妙的技巧，但是你能想象一个代码库：在选项[This]和Option [That]遍布整个地方，以及随后的所有匹配块？**我想说谨慎使用Option。**

## 5. Unit

这是另一个容易使用的。

Unit是不返回任何值的方法的类型。

听起来有点熟？

这就像Java中的一个void返回类型。

这是一个例子：

```scala
scala> def doThreeTimes(fn: (Int) => Unit) = {
     |   fn(1); fn(2); fn(3);
     | }
doThreeTimes: ((Int) => Unit)Unit

scala> doThreeTimes(println)
1
2
3

scala> def specialPrint(num: Int) = {
     |    println(">>>" + num + "<<<")
     | }
specialPrint: (Int)Unit

scala> doThreeTimes(specialPrint)
>>>1<<<
>>>2<<<
>>>3<<<
```

在doThreeTimes的定义中，我们指定该方法使用一个名为fn的参数，其类型为（Int）=> Unit。这意味着fn是一个方法，它接受一个Int类型的单个参数和一个Unit的返回类型，这就是说fn不应该像Java void函数一样返回一个值。

是的。这些是Scala中的“虚无”项目。如果你知道更多，请发表评论！当你拿起Scala的时候，有很多东西需要学习，但作为回报，你会得到一种令人难以置信的表达和简洁的语言。

**参考：**

<http://www.nickknowlson.com/blog/2013/03/31/representing-empty-in-scala/>

[https://sanaulla.info/2009/07/12/nothingness-2/](http://www.nickknowlson.com/blog/2013/03/31/representing-empty-in-scala/)

<https://my.oschina.net/seekerlee/blog/286969>