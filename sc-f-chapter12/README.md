#  第十二篇: 断路器监控(Hystrix Dashboard)(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041113
https://www.cnblogs.com/duanxz/p/7525443.html

本文出自方志朋的博客

Hystrix Dashboard简介
---
在微服务架构中为例保证程序的可用性，防止程序出错导致网络阻塞，出现了断路器模型。
>断路器的状况反应了一个程序的可用性和健壮性，它是一个重要指标。Hystrix Dashboard是作为断路器状态的一个组件，提供了数据监控和友好的图形化界面。

项目配置
---

* 创建一个eureka-server工程，用作服务注册中心
    * 正常配置eureka即可
* 创建service-hi工程：
    * pom配置 
        * 添加spring-cloud-starter-netflix-eureka-client依赖
        * 添加spring-cloud-starter-netflix-hystrix依赖
        * 添加spring-cloud-starter-netflix-hystrix-dashboard依赖
        * 添加spring-boot-starter-actuator依赖
    * application.yml配置
        * 配置server端口
        * 配置application名称
        * 添加eureka配置：eureka.client.serviceUrl
        * 添加endpoints配置：management:endpoints
    * Application配置
        * 添加@EnableEurekaClient注解
        * 添加@EnableCircuitBreaker注解
        * 添加@EnableHystrix注解，开启断路器
        * 添加@EnableHystrixDashboard注解，开启HystrixDashboard
    * 在需要的接口上加@HystrixCommand断路器注解
* 测试：
    * 启动eureka-server和service-hi
    * 在浏览器打开：http://localhost:8762/hi,可以看到：
        >hi forezp ,i am from port:8762
    * 在浏览器打开：http://localhost:8762/actuator/hystrix.stream,可以看到数据监听
    * 在浏览器打开：http://localhost:8762/hystrix，配置参数：
        ![ic_hystrix_config.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter12/doc/ic_hystrix_config.png?raw=true)
        * Url：http://localhost:8762/actuator/hystrix.stream
        * Delay：2000
        * Title：miya
        * 配置好就可以看到Hystrix Dashboard图形化界面
        ![ic_hystrix_board.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter12/doc/ic_hystrix_board.png?raw=true)

python多线程请求接口，查看Hystrix Dashboard监控：
---
* 先配置idea的python环境
![ic_req_python.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter12/doc/ic_req_python.png?raw=true)
* 运行test.py程序即可
![ic_req_result.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter12/doc/ic_req_result.png?raw=true)

  
