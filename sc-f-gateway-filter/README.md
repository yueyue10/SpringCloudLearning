#  第十五篇: Spring Cloud Gateway 之Filter篇

参考：
https://blog.csdn.net/forezp/article/details/85057268

本文出自方志朋的博客

```
刚开始项目运行不起来，最后找到原因：项目的maven配置有问题，重写配置maven路径后可以正常运行。
需要把项目中的AbstractChangeRequestUriGatewayFilterFactory重命名，不然代码依赖到这里就报错了。
```
上一篇讲到：Predict决定了请求由哪一个路由处理，在路由处理之前，需要经过“pre”类型的过滤器处理，处理返回响应之后，可以由“post”类型的过滤器处理。

filter的作用和生命周期
---

由filter工作流程点，可以知道filter有着非常重要的作用。
>在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，

>在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等。

首先需要弄清一点为什么需要网关这一层，这就不得不说下filter的作用了。

### 一、filter的作用

当我们有很多个服务时，比如下图中的user-service、goods-service、sales-service等服务，客户端请求各个服务的Api时，每个服务都需要做相同的事情，比如鉴权、限流、日志输出等。

对于这样重复的工作，有没有办法做的更好，答案是肯定的。

>在微服务的上一层加一个全局的权限控制、限流、日志输出的Api Gateway服务，然后再将请求转发到具体的业务服务层。
这个Api Gateway服务就是起到一个服务边界的作用，外接的请求访问系统，必须先通过网关层。

![ic_filter_index.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-filter/ic_filter_index.png?raw=true)

### 二、生命周期

Spring Cloud Gateway同zuul类似，有“pre”和“post”两种方式的filter。客户端的请求先经过“pre”类型的filter，然后将请求转发到具体的业务服务，比如上图中的user-service，收到业务服务的响应之后，再经过“post”类型的filter处理，最后返回响应到客户端。

![ic_filter_lifecycle.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-filter/ic_filter_lifecycle.png?raw=true)

>在Spring Cloud Gateway中，filter从作用范围可分为另外两种，一种是针对于单个路由的gateway filter，它在配置文件中的写法同predict类似；另外一种是针对于所有路由的global gateway filer。


filter的两种类型：
---

### 1.gateway filter
过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。过滤器可以限定作用在某些特定请求路径上。 Spring Cloud Gateway包含许多内置的GatewayFilter工厂。

GatewayFilter工厂同上一篇介绍的Predicate工厂类似，都是在配置文件application.yml中配置，遵循了约定大于配置的思想，

**只需要在配置文件配置GatewayFilter Factory的名称，而不需要写全部的类名**，比如AddRequestHeaderGatewayFilterFactory只需要在配置文件中写AddRequestHeader，而不是全部类名。

在配置文件中配置的GatewayFilter Factory最终都会相应的过滤器工厂类处理。

Spring Cloud Gateway 内置的过滤器工厂一览表如下：
![ic_filter_factory.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-filter/ic_filter_factory.png?raw=true)

* 此项目中使用了AddRequestHeader、AddResponseHeader、RewritePath的filter
    * 修改application.yml中的spring:profiles:active，然后运行测试
    * 测试filter过滤功能，三种方法：
        * 1.在浏览器请求：http://localhost:8081
        * 2.使用命令行：curl localhost:8081
        * 3.使用Idea的REST Client功能
    * 测试rewritepath_route：请求url测试：http://localhost:8081/foo/forezp
* 自定义过滤器：RequestTimeFilter
    * Ordered中的int getOrder()方法是来给过滤器设定优先级别的，值越大则优先级越低
    * filter(ServerWebExchange exchange, GatewayFilterChain chain)方法中的exchange和chain分别代表“pre”和"post"过滤器
    * 在Application的customerRouteLocator方法里面使用RequestTimeFilter，并加@Bean注解
    * 测试此功能请求url：http://localhost:8081/customer/123，服务后台console输出日志：/customer/123: 858ms
* 自定义过滤器工厂RequestTimeGatewayFilterFactory(~~这个还没有测试~~)
    * 创建自定义过滤器工厂类后，就可以在配置文件中配置过滤器了。
    * 具体使用请参考[方志朋博客][1]
    
### 2.gateway filter

>Spring Cloud Gateway根据作用范围划分为GatewayFilter和GlobalFilter，二者区别如下：
>
>>GatewayFilter : 需要通过spring.cloud.routes.filters 配置在具体路由下只作用在当前路由上,或通过spring.cloud.default-filters配置在全局，作用在所有路由上
>
>>GlobalFilter : 全局过滤器，不需要在配置文件中配置，作用在所有的路由上，最终通过GatewayFilterAdapter包装成GatewayFilterChain可识别的过滤器。
>它为请求业务以及路由的URI转换为真实业务服务的请求地址的核心过滤器，不需要配置，系统初始化时加载，并作用在每个路由上。

Spring Cloud Gateway框架内置的GlobalFilter如下：
![ic_filter_global.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-gateway-filter/ic_filter_global.png?raw=true)

* 自定义GlobalFilter：TokenFilter
    * 在Application中使用tokenFilter方法使用TokenFilter，加上@Bean注解
    * 启动服务后，请求所有接口都需要添加token参数，否则日志会输出：token is empty...

[1]:https://blog.csdn.net/forezp/article/details/85057268
