# 微服务运行数据上报

## 功能描述

微服务可以将自己的运行数据上报给Dashboard服务，在公有云上查看仪表盘数据、分布式事务数据等。该章节是描述如何启用微服务数据上报功能。

## 配置参考

* 增加依赖关系\(pom.xml\)

引入必要的jar包。

```
<dependency>
　　<groupId>com.huawei.paas.cse</groupId>
　　<artifactId>cse-handler-cloud-extension</artifactId>
</dependency>
```

* 配置handler

仪表盘数据依赖于bizkeeper-provider和bizkeeper-consumer。

```yaml
servicecomb:
  handler:
    chain:
      Provider:
        default: auth-provider,qps-flowcontrol-provider,bizkeeper-provider
      Consumer:
        default: auth-consumer,qps-flowcontrol-consumer,loadbalance,fault-injection-consumer,bizkeeper-consumer
```