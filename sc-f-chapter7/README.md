#  第七篇: 高可用的分布式配置中心(Spring Cloud Config)(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041045

本文出自方志朋的博客

当服务实例很多时，都从配置中心读取文件，这时可以考虑将配置中心做成一个微服务，将其集群化，从而达到高可用，架构图如下：
![ic_config.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter7/ic_config.png?raw=true)

**首先说明原项目有个bug：运行config-server报错：找不到主类Application。**
修改{config-server}{pom}{packaging}{pom属性改为jar}

项目配置
---

> 在上一篇项目的基础上进行开发。

* 创建一个eureka-server工程，用作服务注册中心
    * pom配置
        * 添加spring-cloud-starter-netflix-eureka-**server**依赖
        * 添加spring-boot-starter-web依赖
    * application.yml配置
        * 配置server端口
        * 配置eureka
    * Application配置
        * 添加**@EnableEurekaServer**注解
* 改造config-server工程：
    * pom修改 
        * 添加spring-cloud-starter-netflix-eureka-**client**依赖
    * application.properties修改
        * 添加eureka配置：eureka.client.serviceUrl
    * Application修改
        * 添加**@EnableEurekaClient**注解
* 该找config-client工程：
    * pom修改同上
    * bootstrap.properties修改
        * 添加eureka配置：eureka.client.serviceUrl
        * 添加spring.cloud.config.discovery
    * Application修改
        * 添加@EnableEurekaClient注解
        * 添加@RefreshScope注解(~~作用不详，以后再分析~~)
    