#  第四篇:断路器（Hystrix）(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81040990

本文出自方志朋的博客

>在微服务架构中，根据业务来拆分成一个个的服务，服务与服务之间可以相互调用（RPC），在Spring Cloud可以用RestTemplate+Ribbon和Feign来调用。

>为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪。
>
>服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

>为了解决这个问题，业界提出了断路器模型。

断路器简介
---

Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。 
在微服务架构中，一个请求需要调用多个服务是非常常见的，如下图：

![hystrix_1.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter4/hystrix_1.png?raw=true)

较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

![hystrix_2.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter4/hystrix_2.png?raw=true)

断路打开后，可用避免连锁故障，fallback方法可以直接返回一个固定值。


项目配置
---

> 在上一篇ribbon和feign项目的基础上进行完善

* 修改service-ribbon项目
    * pom配置
        * 添加spring-cloud-starter-netflix-hystrix依赖
    * 代码编写：
        * Application代码修改：
            * 加入@EnableHystrix注解开启Hystrix
        * service层代码修改：
            * 在HelloService类里面的hiService方法上加@HystrixCommand()注解
                * @HystrixCommand(fallbackMethod = "hiError")注解对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法
                * 编写hiError熔断方法，此处直接返回一个字符串"hi,"+name+",sorry,error!"。
* 测试service-ribbon断路器功能
    * 启动eureka-service、service-hi、service-ribbon服务
    * 在浏览器上访问http://localhost:8764/hi?name=forezp进行测试,浏览器显示下面内容：
        > hi forezp,i am from port:8762
    * 此时如果我们关闭service-hi服务，再访问http://localhost:8764/hi?name=forezp，浏览器会显示：
        > hi ,forezp,orry,error!
    * 说明当service-hi 服务不可用的时候，service-ribbon调用 service-hi的API接口时，会执行快速失败，直接返回一组字符串，而不是等待响应超时，这很好的控制了容器的线程阻塞。
        ```
        需要先启动eureka-service服务，否则断路器不管用。
        ```
* 修改service-feign项目
    ```
    Feign是自带断路器的，在D版本的Spring Cloud之后，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：
    > feign.hystrix.enabled=true
    ``` 
    * 在SchedualServiceHi接口的注解中加上fallback的指定类即可
        * 编写fallback指定类SchedualServiceHiHystric代码：
            * 实现SchedualServiceHi接口，使用@Component注解
            * 实现接口里面的方法，就是断路错误返回的方法。
* 测试service-feign断路器功能
    * 启动eureka-service、service-feign服务
    * 在浏览器上访问http://localhost:8765/hi?name=forezp进行测试,浏览器显示下面的内容：
        > sorry forezp
    * 启动service-hi工程，再次访问，浏览器显示：
        > hi forezp,i am from port:8762
    * 这证明断路器起到作用了。
      
     
