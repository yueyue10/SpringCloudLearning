#  第十八篇: spring cloud gateway之服务注册与发现

参考：
https://blog.csdn.net/forezp/article/details/85210153

本文出自方志朋的博客

前面我们学习了Spring Cloud Gateway的Predict（断言）、Filter（过滤器），大家对Spring Cloud Gateway有初步的认识，其中在对服务路由转发的这一块，在之前的文章是采用硬编码的方式进行路由转发。

>这篇文章以案例的形式来讲解Spring Cloud Gateway如何配合服务注册中心进行路由转发。

项目配置：
---

涉及到了三个工程， 分别为注册中心eureka-server、服务提供者service-hi、 服务网关service-gateway，如下：

| 工程名	               | 	        端口	        |        作用	           |
| --------                 |          :----:            |      :----:              |
| eureka-server	           |         8761	            |   注册中心eureka server   |
| service-hi	           |         8762	            |   服务提供者 eurka client |
| service-gateway		   |         8081		        |   路由网关 eureka client  |

这三个工程中，其中service-hi、service-gateway向注册中心eureka-server注册。

用户的请求首先经过service-gateway，根据路径由gateway的predict 去断言进到哪一个 router， router经过各种过滤器处理后，最后路由到具体的业务服务，比如 service-hi。如图：

![ic_cloud.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-cloud/ic_cloud.png?raw=true)

### service-gateway配置：

* pom配置：
    * 添加spring-cloud-starter-gateway依赖
* application.yml配置
    * 配置spring:cloud
        > spring.cloud.gateway.discovery.locator.enabled为true
        ```
        表明gateway开启服务注册和发现的功能。
        并且spring cloud gateway自动根据服务发现为每一个服务创建了一个router，这个router将以服务名开头的请求路径转发到对应的服务。
        ```
        >spring.cloud.gateway.discovery.locator.lowerCaseServiceId
        ```
        是将请求路径上的服务名配置为小写（因为服务注册的时候，向注册中心注册时将服务名转成大写的了），
        比如以/service-hi/*的请求路径被路由转发到服务名为service-hi的服务上。
        ```
    * 配置eureka
* 运行项目，测试gateway路由功能：
    * 在浏览器访问：http://localhost:8081/service-hi/hi?name=1323返回：
        > hi 1323 ,i am from port:8762

#### 使用gateway的Predicate改造项目，实现路由功能：
* 上面的请求需要带上服务器名称{service-hi}，修改gateway的配置：
    * 将gateway.discovery.locator.enabled设置为false
    * 添加一个{predicates-Path}功能的route
        * 配置：routes: predicates:- Path=/demo/**
        * 配置：routes: uri: lb://SERVICE-HI
        ```
        将以/demo/**开头的请求都会转发到uri为lb://SERVICE-HI的地址上,lb://SERVICE-HI即service-hi服务的负载均衡地址
        用StripPrefix的filter 在转发之前将/demo去掉
        ```
* 运行项目，测试gateway路由功能：
    * 在浏览器访问：http://localhost:8081/demo/hi?name=1323返回：
    > hi 1323 ,i am from port:8763
