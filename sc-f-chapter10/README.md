#  第十篇: 高可用的服务注册中心(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041101

本文出自方志朋的博客

服务注册中心Eureka Server，是一个实例。当成千上万个服务向它注册的时候，它的负载是非常高的，这在生产环境上是不太合适的，这篇文章主要介绍怎么将Eureka Server集群化。

### eureka集群
Eureka通过运行多个实例，使其更具有高可用性。事实上，这是它默认的熟性，你需要做的就是给对等的实例一个合法的关联serviceurl。

项目配置
---

* 创建一个eureka-server工程，用作服务注册中心
    * pom配置
        * 添加spring-cloud-starter-netflix-eureka-**server**依赖
    * application.yml配置
        * 配置profiles: peer1和peer2
        * 配置使用的环境spring.profiles.active: peer1
            * 启动peer1之后，再修改成peer2再启动peer2
    * Application配置
        * 添加**@EnableEurekaServer**注解
* 创建service-hi工程：
    * pom配置 
        * 添加spring-cloud-starter-netflix-eureka-**client**依赖
    * application.yml配置
        * 配置server端口
        * 配置application名称
        * 添加eureka配置：eureka.client.serviceUrl
    * Application配置
        * 添加**@EnableEurekaClient**注解
* 修改C:\Windows\System32\drivers\etc\hosts，加上以下代码
    ```
    127.0.0.1 peer1
    127.0.0.1 peer2
    ```
* 测试eureka集群：
    * 启动eureka-peer1和eureka-peer2
    * 启动service-hi
    * 在浏览器打开：http://localhost:8761/
    ![ic_peer1.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter10/ic_peer1.png?raw=true)
    * 在浏览器打开：http://localhost:8769/
    ![ic_peer2.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter10/ic_peer2.png?raw=true)

项目的架构图：
---
![ic_eureka.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter10/ic_eureka.png?raw=true)

  
