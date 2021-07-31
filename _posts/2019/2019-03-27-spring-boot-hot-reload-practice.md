---
layout: post
title: Spring boot hot reload practice
categories: [Java, SpringBoot]
description: Spring boot hot reload practice
keywords: [Java, SpringBoot]
---

# 跬步2 Spring boot hot reload practice


前几天曾经和人杰讨论了一下go相对于java的优点.  
其中一点就是go可以丢可执行文件,java则至少需要jvm启动.  

# Notice
本文的受众对象是使用Spring-boot的后端技术,IDE是Intellij.  
Eclipse也同样可以,限于篇幅请自行尝试  
# What is the case
我们程序员在工作中,经常遇到这样的case:  
code写完,在本地把工程run起来自测一下.  

这时,如果发现有个小问题,我们怎么办?  
通常情况,我们如下操作:  
1. 修改代码  
2. 保存  
3. stop then rebuild then restart你的工程  
4. 验证行为,如果还有问题,回到1  

# Why need do sth
1,2,4的行为都是必须,没什么可说的.  
问题是3有的时候耗时比较久.  
具体多久呢?同样的环境下,看工程,工程越复杂越久.  
比如我做《Spring类型参数枚举化实践》分享的demo工程(以下均简称demo工程)耗时3-4s,payment的工程要本地35秒左右,sit环境300s是寻常事.  
payment本地运行的log  
```
{payment-no-account-base} 2019-02-02 14:38:41.013 INFO [main] c.e.c.p.NoAccountBaseApplication [StartupInfoLogger.java:57] tradeId-  span- - Started NoAccountBaseApplication in 33.207 seconds (JVM running for 34.241)
```
demo工程本地运行的log
```
2019-02-02 14:40:40.307  INFO 17052 --- [  restartedMain] c.e.coreservices.demo.DemoApplication    : Started DemoApplication in 3.718 seconds (JVM running for 4.885)
```
如果我们反复进行几次验证,就更烦了,就违背了程序员美德之二——急躁  
所以我们需要在3上刮出点"油水"  
# How 
仔细观察上面步骤的3`stop then rebuild then restart`,可以再分3步
1. stop
2. rebuild
3. restart  

2是必须的,我们能不能在1,3上省出点时间呢?
下面是我的实践,欢迎拍砖  
## 1 Devtools
pom文件加dependency
```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-devtools</artifactId>
</dependency>
```
## 2 IDE
再次强调,这里针对的是Intellij  
### 2.1 Enable auto build
Preferences->Build,Execution,Deployment->Compiler  
Enable `build project automatically`(click the checkbox).  
注意后面有一行提示"only works while not running/debugging"  
所以还需要下一步  
### 2.2 Enable auto make while running
Press: ctrl + shift + A (For Mac ⌘ + shift + A)  
Type: Registry  
Find the key `compiler.automake.allow.when.app.running`  
Enable it(click the checkbox)  
我之前2018.3.2的版本设置到这里就ok了.  
是升级到3.3(新出的3.4一样)版本以后,hot reload不行了,我比较菜不清楚具体原因.  
这时候进行下一步  
### 2.3 Set run configurations
Run->Edit configurations->Templates->Spring boot  
在`Configuration`tab的`Spring boot`block里的  
* `on 'Update' action`
* `on frame deactivation`

两项设置为`Update classes and resources`(默认是`Do nothing`)
Click `Apply` then `OK`  
### 2.4 Restart your Intellij
# Result
以下是启动工程后,再修改代码的console log  
Demo工程
```
2019-02-02 15:46:11.392  INFO 17153 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-02-02 15:46:11.460  INFO 17153 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-02-02 15:46:11.461  INFO 17153 --- [  restartedMain] c.e.coreservices.demo.DemoApplication    : Started DemoApplication in 0.874 seconds (JVM running for 2343.055)
2019-02-02 15:46:11.465  INFO 17153 --- [  restartedMain] .ConditionEvaluationDeltaLoggingListener : Condition evaluation unchanged
2019-02-02 15:46:12.607  INFO 17153 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-02-02 15:46:12.607  INFO 17153 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-02-02 15:46:12.617  INFO 17153 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 10 ms
```
重启时间0.874s,比之前3.7s快了一些  
Payment工程
```
{payment-no-account-base} 2019-02-02 16:02:49.805 INFO [restartedMain] c.e.c.p.NoAccountBaseApplication [StartupInfoLogger.java:57] tradeId-  span- - Started NoAccountBaseApplication in 20.977 seconds (JVM running for 85.024)
```
20.9s,比之前的33.2s还是快一点  
最重要的是不需要手动点击`Stop and Restart`了  
观察日志,其实整个还是reload了一遍,不知道还有没有更好的办法,欢迎讨论
