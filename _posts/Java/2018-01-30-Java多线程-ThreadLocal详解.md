## 1. 适用场景

如上文所述，ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

对于第一点，每个线程拥有自己实例，实现它的方式很多。例如可以在线程内部构建一个单独的实例。ThreadLocal 可以以非常方便的形式满足该需求。

对于第二点，可以在满足第一点（每个线程有自己的实例）的条件下，通过方法间引用传递的形式实现。ThreadLocal 使得代码耦合度更低，且实现更优雅。

## 2. 场景

### 2.1管理Connection

**最典型的是管理数据库的Connection：**当时在学JDBC的时候，为了方便操作写了一个简单数据库连接池，需要数据库连接池的理由也很简单，频繁创建和关闭Connection是一件非常耗费资源的操作，因此需要创建数据库连接池～

那么，数据库连接池的连接怎么管理呢？？我们交由ThreadLocal来进行管理。为什么交给它来管理呢？？ThreadLocal能够实现**当前线程的操作都是用同一个Connection，保证了事务！**

当时候写的代码：

```java
public class DBUtil {
    //数据库连接池
    private static BasicDataSource source;

    //为不同的线程管理连接
    private static ThreadLocal<Connection> local;


    static {
        try {
            //加载配置文件
            Properties properties = new Properties();

            //获取读取流
            InputStream stream = DBUtil.class.getClassLoader().getResourceAsStream("连接池/config.properties");

            //从配置文件中读取数据
            properties.load(stream);

            //关闭流
            stream.close();

            //初始化连接池
            source = new BasicDataSource();

            //设置驱动
            source.setDriverClassName(properties.getProperty("driver"));

            //设置url
            source.setUrl(properties.getProperty("url"));

            //设置用户名
            source.setUsername(properties.getProperty("user"));

            //设置密码
            source.setPassword(properties.getProperty("pwd"));

            //设置初始连接数量
            source.setInitialSize(Integer.parseInt(properties.getProperty("initsize")));

            //设置最大的连接数量
            source.setMaxActive(Integer.parseInt(properties.getProperty("maxactive")));

            //设置最长的等待时间
            source.setMaxWait(Integer.parseInt(properties.getProperty("maxwait")));

            //设置最小空闲数
            source.setMinIdle(Integer.parseInt(properties.getProperty("minidle")));

            //初始化线程本地
            local = new ThreadLocal<>();


        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws SQLException {
        
        if(local.get()!=null){
            return local.get();
        }else{
        
            //获取Connection对象
            Connection connection = source.getConnection();
    
            //把Connection放进ThreadLocal里面
            local.set(connection);
    
            //返回Connection对象
            return connection;
        }

    }

    //关闭数据库连接
    public static void closeConnection() {
        //从线程中拿到Connection对象
        Connection connection = local.get();

        try {
            if (connection != null) {
                //恢复连接为自动提交
                connection.setAutoCommit(true);

                //这里不是真的把连接关了,只是将该连接归还给连接池
                connection.close();

                //既然连接已经归还给连接池了,ThreadLocal保存的Connction对象也已经没用了
                local.remove();

            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }


}
复制代码
```

同样的，Hibernate对Connection的管理也是采用了相同的手法(使用ThreadLocal，当然了Hibernate的实现是更强大的)～

### 2.2 避免一些参数传递

**避免一些参数的传递的理解**可以参考一下Cookie和Session：

- 每当我访问一个页面的时候，浏览器都会帮我们从硬盘中找到对应的Cookie发送过去。
- 浏览器是十分聪明的，不会发送别的网站的Cookie过去，只带当前网站发布过来的Cookie过去

浏览器就相当于我们的ThreadLocal，它仅仅会发送我们当前浏览器存在的Cookie(ThreadLocal的局部变量)，不同的浏览器对Cookie是隔离的(Chrome,Opera,IE的Cookie是隔离的【在Chrome登陆了，在IE你也得重新登陆】)，同样地：线程之间ThreadLocal变量也是隔离的....

那上面避免了参数的传递了吗？？其实是避免了。Cookie并不是我们手动传递过去的，并不需要写``来进行传递参数...

在编写程序中也是一样的：日常中我们要去办理业务可能会有很多地方用到身份证，各类证件，每次我们都要掏出来很麻烦

```java
    // 咨询时要用身份证，学生证，房产证等等....
    public void consult(IdCard idCard,StudentCard studentCard,HourseCard hourseCard){

    }

    // 办理时还要用身份证，学生证，房产证等等....
    public void manage(IdCard idCard,StudentCard studentCard,HourseCard hourseCard) {

    }

    //......

复制代码
```

而如果用了ThreadLocal的话，ThreadLocal就相当于一个机构，ThreadLocal机构做了记录你有那么多张证件。用到的时候就不用自己掏了，问机构拿就可以了。

在咨询时的时候就告诉机构：来，把我的身份证、房产证、学生证通通给他。在办理时又告诉机构：来，把我的身份证、房产证、学生证通通给他。...

```java
    // 咨询时要用身份证，学生证，房产证等等....
    public void consult(){

        threadLocal.get();
    }

    // 办理时还要用身份证，学生证，房产证等等....
    public void takePlane() {
        threadLocal.get();
    }

复制代码
```

这样是不是比自己掏方便多了。



## 3. ThreadLocal 实现原理

![image-20200302174437133](https://tva1.sinaimg.cn/large/00831rSTgy1gcfq5bze2hj314c0loamn.jpg)

### 3.1 ThreadLocal的实现

`ThreadLocal`的实现是这样的：每个`Thread` 维护一个 `ThreadLocalMap` 映射表，这个映射表的 `key` 是 `ThreadLocal`实例本身，`value` 是真正需要存储的 `Object`。

也就是说 `ThreadLocal` 本身并不存储值，它只是作为一个 `key` 来让线程从 `ThreadLocalMap` 获取 `value`。值得注意的是图中的虚线，表示 `ThreadLocalMap` 是使用 `ThreadLocal` 的弱引用作为 `Key` 的，弱引用的对象在 GC 时会被回收。



## 4. 为什么会内存泄漏

`ThreadLocalMap`使用`ThreadLocal`的弱引用作为`key`，如果一个`ThreadLocal`没有外部强引用来引用它，那么系统 GC 的时候，这个`ThreadLocal`势必会被回收，这样一来，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话，这些`key`为`null`的`Entry`的`value`就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value`永远无法回收，造成内存泄漏。

其实，`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。

但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用`static`的`ThreadLocal`，延长了`ThreadLocal`的生命周期，可能导致的内存泄漏（参考[ThreadLocal 内存泄露的实例分析](http://blog.xiaohansong.com/2016/08/09/ThreadLocal-leak-analyze/)）。
- 分配使用了`ThreadLocal`又不再调用`get()`,`set()`,`remove()`方法，那么就会导致内存泄漏。

### 4.1. 为什么使用弱引用

从表面上看内存泄漏的根源在于使用了弱引用。网上的文章大多着重分析`ThreadLocal`使用了弱引用会导致内存泄漏，但是另一个问题也同样值得思考：为什么使用弱引用而不是强引用？

我们先来看看官方文档的说法：

> To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys.
>  为了应对非常大和长时间的用途，哈希表使用弱引用的 key。

下面我们分两种情况讨论：

- **key 使用强引用**：引用的`ThreadLocal`的对象被回收了，但是`ThreadLocalMap`还持有`ThreadLocal`的强引用，如果没有手动删除，`ThreadLocal`不会被回收，导致`Entry`内存泄漏。
- **key 使用弱引用**：引用的`ThreadLocal`的对象被回收了，由于`ThreadLocalMap`持有`ThreadLocal`的弱引用，即使没有手动删除，`ThreadLocal`也会被回收。`value`在下一次`ThreadLocalMap`调用`set`,`get`，`remove`的时候会被清除。

比较两种情况，我们可以发现：由于`ThreadLocalMap`的生命周期跟`Thread`一样长，如果都没有手动删除对应`key`，都会导致内存泄漏，但是使用弱引用可以多一层保障：**弱引用`ThreadLocal`不会内存泄漏，对应的`value`在下一次`ThreadLocalMap`调用`set`,`get`,`remove`的时候会被清除**。

因此，`ThreadLocal`内存泄漏的根源是：由于`ThreadLocalMap`的生命周期跟`Thread`一样长，如果没有手动删除对应`key`就会导致内存泄漏，而不是因为弱引用。

## 5. ThreadLocal 最佳实践

综合上面的分析，我们可以理解`ThreadLocal`内存泄漏的前因后果，那么怎么避免内存泄漏呢？

- 每次使用完`ThreadLocal`，都调用它的`remove()`方法，清除数据。

在使用线程池的情况下，没有及时清理`ThreadLocal`，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用`ThreadLocal`就跟加锁完要解锁一样，用完就清理。



## 6. 总结

由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，我觉得是这种数据结构导致，会产生内存溢出的问题

Java为了最小化减少内存泄露的可能性和影响，在ThreadLocal的get,set的时候都会清除线程Map里所有key为null的value。所以最怕的情况就是，threadLocal对象设null了，开始发生“内存泄露”，然后使用线程池，这个线程结束，线程放回线程池中不销毁，这个线程一直不被使用，或者分配使用了又不再调用get,set方法，那么这个期间就会发生真正的内存泄露。



链接：https://www.jianshu.com/p/1342a879f523

链接：https://juejin.im/post/5ac2eb52518825555e5e06ee

链接：http://www.jasongj.com/java/threadlocal/