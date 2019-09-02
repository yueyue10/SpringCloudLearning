#  第六篇: 分布式配置中心(Spring Cloud Config)(Finchley版本)

参考：
https://blog.csdn.net/forezp/article/details/81041028

本文出自方志朋的博客

Config简介
---

>在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。

>在Spring Cloud中，有分布式配置中心组件spring cloud config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。

>在spring cloud config 组件中，分两个角色，一是config server，二是config client。


项目配置
---
* 创建主项目
    * pom配置：
        * 确保pom中包含{groupId}、{modules}、{parent}、{properties}、{dependencies}、{dependencyManagement}、{build}
* 创建config-server项目
    * pom配置
        * 添加spring-cloud-config-server依赖
        * 确保已经配置{groupId}、{parent}、{dependencies}
    * application.yml配置
        * 配置server端口：**8888**
        * 配置spring服务名称
        * 配置config.server:
           * 配置git.uri-{此处设置的是本项目的一个目录：config}
           * 配置git.searchPaths
           * 配置label
                ```
                # 配置说明
                spring.cloud.config.server.git.uri：配置git仓库地址
                spring.cloud.config.server.git.searchPaths：配置仓库路径
                spring.cloud.config.label：配置仓库的分支
                spring.cloud.config.server.git.username：访问git仓库的用户名
                spring.cloud.config.server.git.password：访问git仓库的用户密码
                ```
            * 如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写
    * Application代码编写：
        * 加上注解@EnableConfigServer：开启配置服务器的功能
* 在主项目下新建config文件夹，用于存放配置文件
* 创建config-client项目
    * pom配置
        * 添加spring-cloud-starter-config依赖
        * 确保已经配置{groupId}、{parent}、{dependencies}
    * **bootstrap.properties配置**
        * 配置server端口
        * 配置spring服务名称：**config-clt**
        * 配置config服务器属性：spring.cloud.config
           * 配置uri：**http://localhost:8888/**
           * 配置label：
           * 配置profile：**test**
            ```
            spring.cloud.config.label 指明远程仓库的分支
            spring.cloud.config.profile
                dev开发环境配置文件
                test测试环境
                pro正式环境
            spring.cloud.config.uri= http://localhost:8888/ 指明配置服务中心的网址。
            ```
    * Application代码编写：
        * 加上注解@RestController，开启RestApi功能
        * 使用@Value注解引入foo属性
        * 创建hi方法提供{/hi}接口，获取foo值
* 在config文件夹中创建配置属性文件：**config-clt-test.properties**
    * 文件名是config-client中的spring.application.name服务器名称+spring.cloud.config.profile环境名称
    * 在其中加入foo = hello foo version 21
* 启动config-server项目，测试远程配置服务可以使用：
    * 在浏览器地址输入：http://localhost:8888/foo/test返回json信息证明配置服务中心可以从远程程序获取配置信息。
        ```
        http请求地址和资源文件映射如下:
        /{application}/{profile}[/{label}]
        /{application}-{profile}.yml
        /{label}/{application}-{profile}.yml
        /{application}-{profile}.properties
        /{label}/{application}-{profile}.properties
        ```
* 启动config-client项目，测试config-client从config-server可以获取配置属性
    * 在浏览器输入：http://localhost:8882/hi返回下面的内容，证明配置成功。
        > hello foo version 21

#### config-client从config-server获取了foo的属性，而config-server是从git仓库读取的,如下图：

![ic_config.png](https://github.com/yueyue10/SpringCloudLearning/blob/master/sc-f-chapter6/ic_config.png?raw=true)