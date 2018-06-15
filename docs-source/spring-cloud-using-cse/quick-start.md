# 遗留系统快速接入


本章节通过一个实际的案例，说明Spring Cloud应用如何经过少量的配置修改，快速接入CSE。Demo下载地址：[GitHub](https://github.com/huawei-microservice-demo/SpringCloudIntegration)  [码云Gitee](https://gitee.com/hwssjhw/SpringCloudIntegration)

* 原始Spring Cloud应用位于SpringCloudIntegration的子项目：[springcloud-sample](https://github.com/huaweicse/cse-java-chassis-samples/tree/master/springcloud-sample)
* 修改后的Spring Cloud应用位于SpringCloudIntegration的子项目：[springcloud-sample-2-cse](https://github.com/huawei-microservice-demo/SpringCloudIntegration/tree/master/springcloud-sample-2-cse)，该项目已提供Dockerfile和start.sh，可以直接拷贝以便在微服务云应用平台快速构建镜像。

springcloud-sample提供了3个项目：

* eureka-server 提供注册发现能力。
* springcloud-provider 服务提供者，该服务提供了名称为HelloService的REST接口。
* springcloud-consumer 服务消费者，该服务也提供了名称为HelloService的REST接口，其实现通过Feign调用springcloud-provider的REST接口。

修改后的springcloud-sample-2-cse具备如下能力和变化：

* 使用CSE提供的服务中心作为注册发现服务；
* 使用CSE提供的配置中心作为动态配置服务，可以通过配置中心管理公共配置；
* 业务的其他逻辑不发生任何变化，写代码的方式也不发生变化。开发者仍然可以按照原来的开发习惯书写业务代码。

## 接入步骤说明

CSE为Spring Cloud应用提供了非常简单的接入方式，开发者只需要修改依赖关系和少量的配置，就可以启用服务中心和配置中心客户端连接功能，将Spring Cloud应用作为一个CSE的微服务注册到服务中心和使用动态配置能力。

* 参考[环境准备](https://support.huaweicloud.com/qs-cse/cse_qs_0012.html)配置maven setting文件，并将Spring Cloud中对于eureka的依赖换成CSE的依赖。
  Eureka的依赖：开发者一般会使用spring-cloud-starter-eureka。spring-cloud-starter-eureka-server是作为注册服务使用的，替换服务中心后不需要继续使用。

删除如下依赖，以springcloud-sample为例，修改springcloud-provider/pom.xml,springcloud-consumer/pom.xml：
```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

删除如下依赖，以springcloud-sample为例，修改父pom.xml：
```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

增加如下依赖，以springcloud-sample为例，修改父pom.xml：
```
<dependency>
  <groupId>com.huawei.paas.cse</groupId>
  <artifactId>cse-solution-spring-cloud</artifactId>
  <version>2.3.20</version>
</dependency>
```

[可选]eureka-server已不再需要，可以删除，以springcloud-sample为例，修改父pom.xml，只保留springcloud-provider和springcloud-consumer：
```
<modules>
  <module>springcloud-provider</module>
  <module>springcloud-consumer</module>
</modules>
```

* Consumer使用ribbon

springcloud-sample的springcloud-consumer模块使用的Ribbon默认对接Eureka，需要在springcloud-consumer的application.yml中增如下配置：

```
helloprovider:
  ribbon:
    NIWSServerListClassName: org.apache.servicecomb.springboot.starter.discovery.ServiceCombServerList
```
其中：

* <clientId>.ribbon.NIWSServerListClassName: RibbonClient的配置规则，本例中helloprovider是clientId，即服务消费者需要访问的服务提供者的微服务名称。
* org.apache.servicecomb.springboot.starter.discovery.ServiceCombServerList: CSE服务实例清单的维护机制

经过上面步骤，就完成了Spring Cloud应用接入CSE的全部整改。开发者可以将应用打包为容器镜像，在公有云上进行部署。另外，[springcloud-sample-2-cse](https://github.com/huawei-microservice-demo/SpringCloudIntegration/tree/master/springcloud-sample-2-cse)项目已提供Dockerfile和start.sh，可以直接拷贝以便在微服务云应用平台快速构建镜像。


## 体验改造后的服务
本地调试时，需要参照“补充说明”第2点，在application.yml中增加认证信息。

访问服务： http://localhost:7211/hello?name=World

![](/assets/spring-cloud-integration-008)

说明：

部署在华为云上时，请将localhost:7211替换为实际的访问地址。您可以在应用详情页面中，从“概览”中获取“外部访问地址”。

### 服务目录
登录华为云，访问微服务引擎，可以在"微服务管理" > "服务目录" 查看到注册成功的两个服务以及实例信息。

### 动态配置
为了演示，在HelloService中增加了如下接口
```
  @Value(value = "${cse.dynamic.property:null}")
  String value;

  @RequestMapping(method = RequestMethod.GET)
  public String dynamicProperty() {
    String dynamicProperty = DynamicPropertyFactory.getInstance().getStringProperty("cse.dynamic.property", "").get();
    return "@Value is " + value + "; Api read is " + dynamicProperty;
  }
```
并通过微服务引擎的"动态配置"增加配置项，访问 http://localhost:7211/hello/dynamicProperty ，得到如下結果：
```
@Value is property; Api read is property
```
修改配置项的值为其他值，得到
```
@Value is property; Api read is propertyChanged
```
@Value注入的值不会动态变化，通过API获取的值会动态变化。Spring Cloud使用的大量组件，包括Hystrix, Ribbon等都是通过API读取的配置，这些配置项都能够动态读取到。

Spring Cloud还提供了@ConfigurationProperties简化配置，但它的工作原理和@Value以及API都不同，只能够读取到application.yml配置文件中的配置项，不支持动态配置。

### 其他功能
该接入步骤完成了上云的第一步，只能使用服务目录和动态配置功能。经过进一步的改造Spring Cloud应用才能够使用仪表盘、服务治理等功能。

## 补充说明

* 建议开发者在pom.xml中引入依赖的dependencyManagement，以便更好的管理使用的三方件，防止冲突。

dependencyManagement不会往程序里面增加依赖关系，但是可以帮助开发者更好的管理依赖关系，对于解决三方软件冲突非常有用。详细原理描述可以参考“[使用maven管理复杂依赖关系的技巧](http://servicecomb.incubator.apache.org/cn/docs/maven_dependency_management/)"。

```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.huawei.paas.cse</groupId>
                <artifactId>cse-dependency</artifactId>
                <version>2.3.20</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

* 将应用部署在云上虚拟机集群内时，部署平台会对应用增加一些认证关系的配置，并自动获取认证信息(AK/SK)完成安全认证。CSE的服务中心和配置中心在api gateway开放了REST接口，支持开发者在公网环境使用其服务，为开发者的线下开发带来极大的便利。线下使用CSE的服务中心和配置中心，开发者需要在application.yml中增加认证信息，认证信息包含AS/SK，可以从华为云账号的"[我的凭证](https://support.huaweicloud.com/usermanual-iam/zh-cn_topic_0079477318.html)”获取。

```
servicecomb:
  credentials:
    accessKey: your access key
    secretKey: your secret key
    akskCustomCipher: default
```

在上面的步骤中，实际隐含了将服务中心的地址设置为华北区cn-north-1，如果需要使用其他区域的服务中心和配置中心地址，还需要显示的指定地址和区域。

华为云CSE已上线的区域请访问[地区和终端节点](https://developer.huaweicloud.com/endpoint?cse)。此处，以华为云“华东-上海二”区域为例配置如下

```
servicecomb:
  service:
    registry:
      address: https://cse.cn-east-2.myhuaweicloud.com:443
  config:
    client:
      serverUri: https://cse.cn-east-2.myhuaweicloud.com:443
  credentials:
    project: cn-east-2
```

有些开发者需要通过代理服务器访问华为云，也可以通过设置代理来实现：

```
servicecomb:
  proxy:
    enable: true
    host: your proxy server
    port: your proxy server port
    username: user name
    passwd: password for proxy 
```
CSE的配置是分层次的，按照优先级顺序是：

yaml配置文件 < 环境变量 < System Property < 配置中心。

如果开发者不希望将密码信息写入配置文件，也可以通过环境变量或者System Property的方式设置这些配置信息。比如：

java -Dcse.credentials.accessKey=$ACCESS_KEY Application.jar。
