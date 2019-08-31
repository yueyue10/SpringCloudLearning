# 第一篇： 服务的注册与发现Eureka(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81040925

本文出自方志朋的博客

项目配置
---

* 创建一个Maven项目，或者Spring项目
    * 配置主项目的pom：
        * 配置SpringBoot版本：{parent}{spring-boot-starter-parent}
        * 配置SpringCloud版本：{dependencyManagement}{dependencies}{dependency}{spring-cloud-dependencies}
            * 在{properties}{spring-cloud.version}里面编辑SpringCloud版本号
    * 确保pom中包含：{groupId相关}、{parent}、{modules}、{properties}、{dependencyManagement}标签
        * 其他{dependencies}、{build}标签可以放到module的pom里面
* 创建eureka-server项目
    * 配置pom：
        * 配置eureka依赖：spring-cloud-starter-netflix-eureka-server
        * 添加parent为主项目，artifactId：sc-f-chapter1
        * 确保pom中包含{groupId相关}、{parent}、{dependencies}
    * 配置Application：
        * 为Application添加@EnableEurekaServer注解
    * 配置application.yml
        * 配置server端口
        * 配置eureka
        * 配置spring项目名称
* 创建service-hi项目
    * 配置pom：
        * 配置eureka依赖：spring-cloud-starter-netflix-eureka-server
        * 配置web依赖：spring-boot-starter-web
        * 添加parent为主项目，artifactId：sc-f-chapter1
        * 配置pom中包含{groupId相关}、{parent}、{dependencies}
    * 配置Application：
            * 为Application添加@EnableEurekaServer注解
            * 为Application添加@RestController注解
            * 在Application中添加一个{/hi}的api：home()方法
    * 配置application.yml
        * 配置server端口
        * 配置spring项目名称
        * 配置eureka
* 启动eureka-server服务，再启动service-hi服务，可以在：http://localhost:8761/里面看到{SERVICE-HI}
    * 关闭service-hi服务，在eureka里面还可以看到。~~（这应该十个问题吧？）~~