#  第三篇: 服务消费者（Feign）(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81040965

本文出自方志朋的博客

Feign简介
---

* Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。
* 使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，
* 可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。
* Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

简而言之：
* Feign 采用的是基于接口的注解
* Feign 整合了ribbon，具有负载均衡的能力
* 整合了Hystrix，具有熔断的能力

项目配置
---

> 在上一篇eureka项目基础上，启动service-hi：8762和service-hi：8763。
>> 这时你会发现：service-hi在eureka-server注册了2个实例，这就相当于一个小的集群。

* 创建service-feign项目
    * pom配置
        * 添加spring-cloud-starter-openfeign依赖
        * 确保已经配置{groupId}、{parent}、{dependencies}
    * application.yml配置
        * 配置eureka
        * 配置server端口
        * 配置spring服务名称
    * 代码编写：
        * Application代码编写：
            * 加入@EnableEurekaClient和@EnableDiscoveryClient注解
            * 加入@EnableFeignClients注解
        * service层代码编写：
            * 创建SchedualServiceHi接口
            * 使用@FeignClient(value = "service-hi")注解，来指定调用哪个服务。
            * 创建sayHiFromClientOne方法调用service-hi服务的{/hi}接口
        * controller层代码编写：
            * 创建HiController类，使用@RestController注解
            * 引入SchedualServiceHi，使用@Autowired注解
            * 创建sayHi方法提供ResetApi，使用SchedualServiceHi实现具体逻辑。
* 在浏览器上多次访问http://localhost:8765/hi?name=forezp进行测试
    * 浏览器交替显示下面内容
        > hi forezp,i am from port:8762
        >
        > hi forezp,i am from port:8763
    * 这说明当我们已经实现了均衡。