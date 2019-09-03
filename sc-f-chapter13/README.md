#  第十三篇: 断路器聚合监控(Hystrix Turbine)(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041125

本文出自方志朋的博客

上一篇介绍了利用Hystrix Dashboard去监控断路器的Hystrix command。
>当我们有很多个服务的时候，这就需要聚合所有服务的Hystrix Dashboard的数据了。这就需要用到Spring Cloud的另一个组件了，即Hystrix Turbine。

### Hystrix Turbine简介
看单个的Hystrix Dashboard的数据并没有什么多大的价值，要想看这个系统的Hystrix Dashboard数据就需要用到Hystrix Turbine。

>Hystrix Turbine将每个服务Hystrix Dashboard数据进行了整合。Hystrix Turbine的使用非常简单，只需要引入相应的依赖和加上注解和配置就可以了。

项目配置
---

>在上一篇的基础上进行完善

* 创建一个和service-hi相同的服务：service-lucy
    * pom配置
        * 添加spring-cloud-starter-netflix-hystrix-dashboard
        * 添加spring-cloud-starter-netflix-hystrix
        * 添加spring-boot-starter-actuator等
    * Application配置
        * 添加@EnableHystrix
        * 添加@EnableHystrixDashboard
        * 添加@EnableCircuitBreaker
        * 同7个
    * application.yml配置：
        * 添加management配置
* 创建service-turbine项目，整合多个web服务的Hystrix
    * pom配置
        * 配置同上面的依赖
        * 添加spring-cloud-starter-netflix-turbine依赖
    * Application配置
        * 添加@EnableTurbine注解
    * application.yml配置
        * 除了配置management
        * 添加turbine配置
            * 配置需要整合的web服务名称：turbine:app-config
            * 配置其他参数(~~不是很明白，以后再看~~)
* 测试turbine监控
    * 在浏览器访问：http://localhost:8764/turbine.stream可以看到监控数据
    * 在浏览器访问：http://localhost:8764/hystrix
        * Url：http://localhost:8764/turbine.stream
        * Delay：2000
        * Title：turbine-console
    * 也可以使用test.py测试多线程网络请求
    
效果图如下：
---

这里有个问题(~~如果没有请求，界面不知为何就不刷新了~~)
![ic_turbine.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter13/ic_turbine.png?raw=true)

