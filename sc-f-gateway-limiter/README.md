#  第十七篇: Spring Cloud Gateway 之限流篇

参考：
https://blog.csdn.net/forezp/article/details/85081162
https://www.cnblogs.com/cjsblog/p/9379516.html
https://www.jb51.net/article/144287.htm

本文出自方志朋的博客

在高并发的系统中，往往需要在系统中做限流，一方面是为了防止大量的请求使服务器过载，导致服务不可用，另一方面是为了防止网络攻击。

常见的限流方式，比如Hystrix适用线程池隔离，超过线程池的负载，走熔断的逻辑。在一般应用服务器中，比如tomcat容器也是通过限制它的线程数来控制并发的；也有通过时间窗口的平均速度来控制流量。常见的限流纬度有比如通过Ip来限流、通过uri来限流、通过用户访问频次来限流。

一般限流都是在网关这一层做，比如Nginx、Openresty、kong、zuul、Spring Cloud Gateway等；也可以在应用层通过Aop这种方式去做限流。

令牌桶算法
---

![ic_limiter.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-limiter/ic_limiter.png?raw=true)

实现思路：可以准备一个队列，用来保存令牌，另外通过一个线程池定期生成令牌放到队列中，每来一个请求，就从队列中获取一个令牌，并继续执行。

Spring Cloud Gateway限流
---

>在Spring Cloud Gateway中，有Filter过滤器，因此可以在“pre”类型的Filter中自行实现上述三种过滤器。

> 但是限流作为网关最基本的功能，Spring Cloud Gateway官方就提供了RequestRateLimiterGatewayFilterFactory这个类，适用Redis和lua脚本实现了令牌桶的方式。

#### 在Spring Cloud Gateway中使用内置的限流过滤器工厂来实现限流。
* pom配置：添加spring-cloud-starter-gateway依赖、添加spring-boot-starter-data-redis-reactive依赖
* application.yml配置：
    * 配置spring:cloud:gateway
        * 配置{route}{filters}{RequestRateLimiter}
        * 该过滤器需要配置三个参数：
            * burstCapacity，令牌桶总容量。
            * replenishRate，令牌桶每秒填充平均速率。
            * key-resolver，用于限流的键的解析器的 Bean 对象的名字。它使用 SpEL 表达式根据#{@beanName}从 Spring 容器中获取 Bean 对象。
    * 配置spring:cloud:redis
        * 配置{host}、{port}、{database}
    * 配置Application
        * 可以通过KeyResolver来指定限流的Key,比如我们需要根据用户来做限流，IP来做限流等等。
        * 将这个类的Bean注册到Ioc容器中。
        * 具体可参考：[详解Spring Cloud Gateway 限流操作][1]    
* 使用test.py测试Gateway限流
    * 先运行Application程序
    * 再运行test.py程序，每0.1s请求一次http://localhost:8081接口，发现一部分请求被过滤掉了

[1]:https://www.jb51.net/article/144287.htm