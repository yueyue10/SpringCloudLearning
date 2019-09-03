#  第八篇: 消息总线(Spring Cloud Bus)(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041062
https://blog.csdn.net/Anenan/article/details/85134208
https://blog.csdn.net/wtdm_160604/article/details/83720391

本文出自方志朋的博客

项目配置
---

> 在上一篇项目的基础上进行开发。

* 改造config-server工程：
    * pom修改 
        * 添加spring-cloud-starter-bus-amqp依赖
* 改造config-client工程：
    * pom修改
        * 添加spring-cloud-starter-bus-amqp依赖
        * 添加spring-boot-starter-actuator依赖
    * bootstrap.properties修改
        * 添加bus配置：spring.cloud.bus
    * Application修改
        * 添加@RefreshScope注解

### 测试发现初次启动项目可以读取到github配置的config参数，但是用bus通知config修改后读取到的参数却没有改变

> 这里还没有找到原因，以后再慢慢找吧。实在找不到了

此时的架构图：
---

![ic_bus.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter8/ic_bus.png?raw=true)
![ic_bus1.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter8/ic_bus1.png?raw=true)