# 使用轻量级容器和Edge Service

前面的例子展示了如何在Spring Cloud中使用CSE的REST框架。切换过程中也进行了一些接口定义的调整，使得接口定义更加规范和简洁。对于一些只提供业务逻辑的服务，可以使用轻量级的运行容器，让业务运行更高效。一般来说，还需要提供一个网关服务，进行认证鉴权和外部接入。

下面的章节中，介绍如何将服务切换为轻量级容器和实现Edge Service。点击可以[下载](https://github.com/huaweicse/cse-java-chassis-samples/tree/master/springcloud-sample-cse-edge)改造后的项目。

## 改造为轻量级的容器

这部分改造非常简单，只需要修改pom依赖，原理介绍在[JAVA应用方式开发步骤](../using-cse-in-spring-boot/java-application.md)提供了介绍。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>spring-boot-starter-transport</artifactId>
</dependency>
```
修改为
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>spring-boot-starter-provider</artifactId>
</dependency>
```

### 性能比较
为了测试性能，开发了performanceclient服务，该服务使用了CSE的metrics能力输出统计数据。由于性能数据和运行环境、参数配置等都非常大, 下面只给出了在windows开发环境同时运行3个服务的性能结果。
改造后：
```
consumer:
  tps     latency(ms) max-latency(ms) operation
  rest.200:
  2799    3.562       14.571          helloconsumer.hello.hello
  2799    3.562       14.571          
```
改造前：
```
consumer:
  tps     latency(ms) max-latency(ms) operation
  rest.200:
  2479    4.021       15.889          helloconsumer.hello.hello
  2479    4.021       15.889        
```
由此看出，改造后性能有一定程度的提升。本测试只是非常粗糙的测试数据，在不同环境和条件下的提升比例会有巨大的差异。加上应用并未调优，这个数据仅供参考。建议开发者在目标环境运行测试程序。


## 增加Edge Service
Edge Service开发非常简单，只需要将项目拷贝过来即可使用，并且可以应用于其他项目中。Edge Service的核心组成是AbstractEdgeDispatcher，通过SPI进行加载，当需要不同的路由规则的时候，通过增加扩展即可实现。SPI根据实现类的getOrder进行优先级选择。

在[本例子](https://github.com/huaweicse/cse-java-chassis-samples/tree/master/springcloud-sample-cse-edge/edge-service)中，使用了缺省的DefaultEdgeDispatcher，经过简单的配置就可以实现强大的路由能力。
```
servicecomb:
  http:
    dispatcher:
      edge:
        default:
          enabled: true
          prefix: api
          withVersion: false
          prefixSegmentCount: 2
```

增加Edge Service以后，可以通过下面的URL访问原来的服务：

```
http://localhost:7118/api/helloconsumer/hello?name=World
```

可以通过[ServiceComb开发指南](https://huaweicse.github.io/servicecomb-java-chassis-doc/zh_CN/edge/by-servicecomb-sdk.html)获取更多关于Edge Service的使用说明。
