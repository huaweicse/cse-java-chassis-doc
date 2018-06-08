# ServiceComb应用接入CSE

CSE Java SDK 100%兼容ServiceComb Java Chassis，并对其进行更加友好的封装，以简化用户业务开发，更加专注于业务逻辑。将ServiceComb Java Chassis部署到CSE，并使用CSE提供的能力，只需要对microservice.yaml进行适当的配置，以及在pom中添加额外的依赖，不涉及任何代码修改。在本章节，介绍如何一键式启用CSE能力。接下来的章节，详细的介绍CSE的每个具体能力的一些细节。

## 一键式配置

CSE提供了cse-solution-service-engine，让基于开源版本开发的应用快速切换为云上应用，直接使用公有云提供的服务目录、灰度发布、服务治理等功能。这里的配置，包括组合适当的组件、配置处理链、配置参数的缺省值等。具体包括：

* 引入相关依赖的组件。包括必须的Handlers、Providers、Transports。
* 增加默认配置项。默认配置包含了处理链、缺省的负载均衡和治理选项等。

对于ServiceComb应用，可以通过修改POM引入cse-solution-service-engine。

增加/修改依赖管理
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.huawei.paas.cse</groupId>
                <artifactId>cse-dependency</artifactId>
                <version>2.3.23</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

引入依赖
```xml
<dependency> 
  <groupId>com.huawei.paas.cse</groupId>  
  <artifactId>cse-solution-service-engine</artifactId> 
</dependency>
```

## 兼容性和开发者建议
cse-solution-service-engine的目的是提供快速、全面的使用CSE能力的组合方案。随着CSE提供的功能的调整和丰富，它会不断的去适配CSE的变更，以期让开发者获得最新的功能体验。CSE倡导持续集成、迭代更新的工作方式，以提高创新效率和提升软件质量。但是这种方式需要业务系统存在一定的机制，能够检测变更和快速针对变更进行修改。比如，业务系统在集成CSE的新版本的时候，能够运行自己的自动化测试用例，来检测变更，并根据CSE提供的变更说明来快速适配新功能。

这种持续集成、迭代更新的理念并不适合所有的场景和用户。对于寻求业务系统稳定，不期望变更行为发生的情况，建议业务系统参考cse-solution-service-engine的做法，创建自己的solution，业务系统依赖自己的solution，然后根据业务节奏，自行调整solution的行为。即使cse-solution-service-engine发生变化，业务系统的行为也不受影响。

这两种策略是应对变更的不同策略，都有利有弊。通常的建议是对于频繁变化的业务，采用第一种方式，加快业务创新速度；对于需要长期稳定不变化的业务，采用第二种方式。

## cse-solution-service-engine的详细内容
cse-solution-service-engine的主要内容是POM和microservice.yaml。 POM引入了必要的依赖，microservice.yaml指定了一些缺省值配置。下面的配置是2.3.23版本的配置。

* POM

POM引入了相关的组件，这样开发者就不需要分析使用CSE需要引入哪些组件，并逐个自行引入。2.3.23版本默认引入了如下组件：

```
<dependencies>
    <dependency>
        <groupId>com.huawei.paas.cse</groupId>
        <artifactId>foundation-auth</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>config-cc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
    </dependency>

    <!-- transports -->
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>edge-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>transport-highway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>transport-rest-vertx</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>transport-rest-servlet</artifactId>
    </dependency>

    <!-- providers -->
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>provider-springmvc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>provider-pojo</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>provider-jaxrs</artifactId>
    </dependency>

    <!-- handlers -->
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>handler-bizkeeper</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>handler-fault-injection</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>handler-flowcontrol-qps</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>handler-loadbalance</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.servicecomb</groupId>
        <artifactId>handler-publickey-auth</artifactId>
    </dependency>
    <dependency>
        <groupId>com.huawei.paas.cse</groupId>
        <artifactId>cse-handler-cloud-extension</artifactId>
    </dependency>
</dependencies>
```

* microservice.yaml

microservice.yaml配置了缺省的处理链，以及一些缺省值。它的配置优先级cse-config-order值为-1，在没有指定配置优先级的情况下，值为0，因此业务自己的microservice.yaml配置缺省拥有更高的优先级。2.3.23版本默认配置如下：

```
cse-config-order: -1
service_description:
  propertyExtendedClass: com.huawei.paas.middleware.MiddlewarePropertyExt

servicecomb:
  handler:
    chain:
      Provider:
        default: auth-provider,qps-flowcontrol-provider,bizkeeper-provider
      Consumer:
        default: auth-consumer,qps-flowcontrol-consumer,loadbalance,fault-injection-consumer,bizkeeper-consumer
  references:
    version-rule: 0+
  loadbalance:
    serverListFilters: darklaunch,zoneaware
    serverListFilter:
      darklaunch:
        className: com.huawei.paas.darklaunch.DarklaunchServerListFilter
      zoneaware:
        className: org.apache.servicecomb.loadbalance.filter.ZoneAwareServerListFilterExt
addressResolver:
  searchDomains: default.svc.cluster.local,svc.cluster.local,cluster.local
  ndots: 1
```

