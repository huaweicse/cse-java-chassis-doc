# 使用服务治理

## 功能描述

服务治理主要涉及“隔离”、“熔断”、“容错”、“限流”、“负载均衡”、“灰度发布”等。CSE的服务治理都是构建在Handlers之上，因此这里的主要工作是引入相关的组件和配置处理链。

## 配置参考

在cse-solution-service-engine里面，已经给出了相关引入模块的列表和处理链。这里只对处理链做一个简单说明。

```yaml
servicecomb:
  handler:
    chain:
      Provider:
        default: auth-provider,qps-flowcontrol-provider,bizkeeper-provider
      Consumer:
        default: auth-consumer,qps-flowcontrol-consumer,loadbalance,fault-injection-consumer,bizkeeper-consumer
```

## 使用故障注入

故障注入主要提供了延时、错误两种类型故障。它在fault-injection-consumer提供。

## 使用灰度发布

灰度发布用于引流。用户可以根据比例、条件将服务请求转发到符合要求的实例上去。比如在升级的场景，将部分请求引流到新版本，部分请求引流到老版本。灰度发布在loadbalance提供，同时需要扩展几个filter，引入下面的依赖和增加配置项。

* 增加依赖关系\(pom.xml\)

```xml
<dependency> 
  <groupId>com.huawei.paas.cse</groupId>  
  <artifactId>cse-handler-cloud-extension</artifactId>
</dependency>
```

* 在Consumer端配置负载均衡\(microservice.yaml）

在负载均衡模块启用了灰度发布的filter。

```
cse:
 loadbalance:
  serverListFilters: darklaunch
  serverListFilter:
    darklaunch:
      className: com.huawei.paas.darklaunch.DarklaunchServerListFilter
```

## 使用调用链

华为云提供了业务无侵入的埋点功能APM。只需要通过华为云部署容器应用，并选择启用调用链监控功能，即可使用调用链服务。