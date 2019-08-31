# 第二篇: 服务消费者（rest+ribbon）(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81040946

本文出自方志朋的博客

ribbon简介
---
* 在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于http restful的。
* Spring cloud有两种服务调用方式，一种是ribbon+restTemplate，另一种是feign。

> ribbon是一个负载均衡客户端，可以很好的控制http和tcp的一些行为。
>
> Feign默认集成了ribbon。


项目配置
---

> 在上一篇eureka项目基础上，启动service-hi：8762和service-hi：8763。
>> 这时你会发现：service-hi在eureka-server注册了2个实例，这就相当于一个小的集群。

* 创建service-ribbon项目
    * pom配置
        * 添加spring-cloud-starter-netflix-ribbon依赖
        * 确保已经配置{groupId}、{parent}、{dependencies}
    * application.yml配置
        * 配置eureka
        * 配置server端口
        * 配置spring服务名称
    * 代码编写：
        * Application代码编写：
            * 加入@EnableEurekaClient和@EnableDiscoveryClient注解
                ```
                @EnableDiscoveryClient和@EnableEurekaClient共同点就是：都是能够让注册中心能够发现，扫描到改服务。
                
                不同点：@EnableEurekaClient只适用于Eureka作为注册中心，@EnableDiscoveryClient 可以是其他注册中心。
                ```
            * 创建restTemplate()方法提供RestTemplate
                * 使用@Bean注解，向程序的ioc注入一个bean
                * 使用@LoadBalanced注解表明这个restRemplate开启负载均衡的功能
        * service层代码编写：
            * 创建HelloService类，使用@Service注解
                * 引入RestTemplate，并使用@Autowired注解。
                * 创建hiService()方法，使用restTemplate来消费service-hi服务的“/hi”接口。
                    ```
                    // 在这里我们直接用的程序名替代了具体的url地址，
                    // 在ribbon中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的url替换掉服务名
                    restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
                    ```
                * 总结就是：通过之前注入ioc容器的restTemplate来消费service-hi服务的“/hi”接口
        * controller层代码编写：
            * 创建HelloControler类，使用@RestController注解
            * 引入HelloService，使用@Autowired注解
            * 创建hello方法提供ResetApi，使用HelloService实现具体逻辑。
* 在浏览器上多次访问http://localhost:8764/hello?name=forezp进行测试
    * 浏览器交替显示下面内容
        > hi forezp,i am from port:8762
        >
        > hi forezp,i am from port:8763
    * 这说明当我们通过调用restTemplate.getForObject(“http://SERVICE-HI/hi?name=”+name,String.class)方法时，已经做了负载均衡，访问了不同的端口的服务实例

项目架构：
---

![ic_ribbon.png](https://github.com/yueyue10/SpringCloudLearning/tree/master/sc-f-chapter2/ic_ribbon.png)