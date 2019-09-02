#  第五篇: 路由网关(zuul)(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041012

本文出自方志朋的博客

在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。一个简答的微服务系统如下图：

![ic_zuul.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter5/ic_zuul.png?raw=true)

在Spring Cloud微服务系统中，一种常见的负载均衡方式是，客户端的请求首先经过负载均衡（zuul、Ngnix），再到达服务网关（zuul集群），然后再到具体的服。

服务统一注册到高可用的服务注册中心集群，服务的所有的配置文件由配置服务管理（下一篇文章讲述），配置服务的配置文件放在git仓库，方便开发人员随时改配置。

Zuul简介
---

Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如/api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

zuul有以下功能：
* Authentication
* Insights
* Stress Testing
* Canary Testing
* Dynamic Routing
* Service Migration
* Load Shedding
* Security
* Static Response handling
* Active/Active traffic management

项目配置
---

* 创建service-zuul项目
    * pom配置
        * 添加spring-cloud-starter-netflix-zuul依赖
        * 确保已经配置{groupId}、{parent}、{dependencies}
    * application.yml配置
        * 配置eureka
        * 配置server端口
        * 配置spring服务名称
        * 配置zuul:
            ```
            # 以/api-a/ 开头的请求都转发给service-ribbon服务
            zuul:
              routes:
                api-a:
                  path: /api-a/**
                  serviceId: service-ribbon
            ```
    * 代码编写：
        * Application代码编写：
            * 加上注解@EnableZuulProxy，开启zuul的功能
* 依次运行这5个工程，测试zuul路由转发
    * 在浏览器上访问http://localhost:8769/api-a/hi?name=forezp浏览器显示：
        > hi forezp ,i am from port:8762>>ribbon
    * 在浏览器上访问http://localhost:8769/api-b/hi?name=forezp浏览器显示：
        > hi forezp ,i am from port:8762>>feign
    * 这说明当我们已经实现了路由转发功能。
* 添加zuul服务过滤：
    * 编写zuul过滤类：MyFilter 
        * 继承自ZuulFilter 类
        * 添加@Component注解
        * 在filterType方法里面返回"pre"字符串
            > filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
            ```
            pre：路由之前
            routing：路由之时
            post： 路由之后
            error：发送错误调用
            ```
        * 在filterOrder方法里面返回0
            > filterOrder：过滤的顺序
        * 在shouldFilter方法里面返回true
            > shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
        * 在run方法里面添加过滤器的逻辑
            > run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。
    * MyFilter类实现类对{token}的过滤，如果不传会报错。
* 依次运行这5个工程，测试zuul服务过滤功能：
    * 在浏览器上访问http://localhost:8769/api-a/hi?name=forezp浏览器显示：
        > token is empty
    * 在浏览器上访问http://localhost:8769/api-b/hi?name=forezp&token=1223浏览器显示：
        > hi forezp ,i am from port:8762>>feign
    * 这说明当我们已经实现了过滤功能。
