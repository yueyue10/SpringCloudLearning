#  第十五篇: Spring Cloud Gateway 之Predict篇

参考：
https://blog.csdn.net/forezp/article/details/84926662

本文出自方志朋的博客

```
上一篇使用curl命令成功的请求，使用其他请求不管用。这次使用idea提供的REST Client测试可以。
```

### Spring Cloud gateway工作流程

在之前的文章的Spring Cloud Gateway初体验中，大家已经对Spring Cloud Gateway的功能有一个初步的认识，网关作为一个系统的流量的入口，有着举足轻重的作用，通常的作用如下：
* 协议转换，路由转发
* 流量聚合，对流量进行监控，日志输出
* 作为整个系统的前端工程，对流量进行控制，有限流的作用
* 作为系统的前端边界，外部流量只能通过网关才能访问系统
* 可以在网关层做权限的判断
* 可以在网关层做缓存

>Spring Cloud Gateway作为Spring Cloud框架的第二代网关，在功能上要比Zuul更加的强大，性能也更好。

>随着Spring Cloud的版本迭代，Spring Cloud官方有打算弃用Zuul的意思。在笔者调用了Spring Cloud Gateway的使用和功能上，Spring Cloud Gateway替换掉Zuul的成本上是非常低的，几乎可以无缝切换。Spring Cloud Gateway几乎包含了zuul的所有功能。

![ic_geteway.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-predicate/ic_geteway.png?raw=true)

注：该图片来自官网

如上图所示，客户端向Spring Cloud Gateway发出请求。 

如果Gateway Handler Mapping确定请求与路由匹配（这个时候就用到predicate），则将其发送到Gateway web handler处理。 

Gateway web handler处理请求时会经过一系列的过滤器链。 过滤器链被虚线划分的原因是过滤器链可以在发送代理请求之前或之后执行过滤逻辑。
 
 先执行所有“pre”过滤器逻辑，然后进行代理请求。 在发出代理请求之后，收到代理服务的响应之后执行“post”过滤器逻辑。这跟zuul的处理过程很类似。
 
 在执行所有“pre”过滤器逻辑时，往往进行了鉴权、限流、日志输出等功能，以及请求头的更改、协议的转换；转发之后收到响应之后，会执行所有“post”过滤器的逻辑，在这里可以响应数据进行了修改，比如响应头、协议的转换等。

在上面的处理过程中，有一个重要的点就是讲请求和路由进行匹配，这时候就需要用到predicate，它是决定了一个请求走哪一个路由。

predicate简介
---

> Predicate来自于java8的接口。Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（比如：与，或，非）。可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。add–与、or–或、negate–非。

Spring Cloud Gateway内置了许多Predict,这些Predict的源码在org.springframework.cloud.gateway.handler.predicate包中，如果读者有兴趣可以阅读一下。现在列举各种Predicate如下图：

![ic_geteway1.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-predicate/ic_geteway1.png?raw=true)

注：图片来自网络

在上图中，有很多类型的Predicate,比如说时间类型的Predicated（AfterRoutePredicateFactory BeforeRoutePredicateFactory BetweenRoutePredicateFactory），

当只有满足特定时间要求的请求会进入到此predicate中，并交由router处理；cookie类型的CookieRoutePredicateFactory，指定的cookie满足正则匹配，才会进入此router;

以及host、method、path、querparam、remoteaddr类型的predicate，每一种predicate都会对当前的客户端请求进行判断，是否满足当前的要求，如果满足则交给当前请求处理。

如果有很多个Predicate，并且一个请求满足多个Predicate，则按照配置的顺序第一个生效。


项目配置
---

* 创建一个geteway功能的项目，和上个项目相同
    * 只是将所有predicate的配置都放到了application.yml中
    * 使用的predicate属性有：
        * After Route Predicate Factory
        * Header Route Predicate Factory
        * Cookie Route Predicate Factory
        * Host Route Predicate Factory
        * Method Route Predicate Factory
        * Path Route Predicate Factory
        * Query Route Predicate Factory
* 测试Predicate配置
    * 修改application.yml中的环境配置：spring:profiles:active=xxx
    * 启动项目
    * 通过REST Client测试功能
    * 具体可参考[方志朋博客][1]

[1]:https://blog.csdn.net/forezp/article/details/84926662




