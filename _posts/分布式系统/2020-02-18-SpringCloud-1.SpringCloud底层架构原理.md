## 1. Spring Cloud整体介绍

如果你基于**Spring Cloud**对外发布一个接口，实际上就是支持**http协议**的，对外发布的就是一个最最普通的**Spring MVC的http接口**

SpringCloud包括：

**Eureka（注册中心）、Ribbon（负载均衡）、Feign、Zuul（网关）**



## 2. Eureka（注册中心）

![Eureka服务注册中心的原理](https://tva1.sinaimg.cn/large/0082zybpgy1gc0alahblyj31iu0qmtcl.jpg)

使用ReadOnly、ReadWrite缓存的作用：优化并发冲突。



## 3. feign

他是对一个接口打了一个注解，他一定会针对这个注解标注的接口生成动态代理，然后你针对feign的动态代理去调用他的方法的时候，此时会在底层生成http协议格式的请求，/order/create?productId=1

底层的话，使用HTTP通信的框架组件，**HttpClient**，**先得使用Ribbon去从本地的Eureka注册表的缓存里获取出来对方机器的列表，然后进行负载均衡，选择一台机器出来，接着针对那台机器发送Http请求过去即可**



## 4. Ribbon（负载均衡）

配置一下不同的请求路径和服务的对应关系，你的请求到了网关，他直接查找到匹配的服务，然后就直接把请求转发给那个服务的某台机器，**Ribbon从Eureka本地的缓存列表里获取一台机器，负载均衡，把请求直接用HTTP通信框架发送到指定机器上去**



## 5. Zuul 



