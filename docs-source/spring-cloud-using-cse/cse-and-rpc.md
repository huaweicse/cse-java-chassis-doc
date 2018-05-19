在上面的章节中，介绍了Spring Cloud如何使用CSE的服务注册发现、动态配置管理等中间件服务。这些操作的基础是Spring RESTful Web Service \(本质上是一个Servlet，即org.springframework.web.servlet.DispatcherServlet\)。CSE作为一个独立的RPC框架实现，可以非常容易集成到Spring Cloud中。通过将Spring RESTful Web Service替换为CSE，可以给开发者带来如下便利：

* 一致的开发体验。使用CSE的SpringMVC模式，可以获得和Spring RESTful Web Service一致的开发体验，包括一样的声明式Annotation，使用RestTemplate进行访问。

* 更好的RPC支持。使用CSE，开发者不需要在客户端使用Feign等组件访问服务，可以直接使用RPC访问，非常灵活。

* 更好的通信性能和协议扩展。

* 完整的开箱即用的服务治理、监控、调用链等功能。达到一键式启用的目的。

本章节仍然基于快速接入的示例，展示改造的步骤，以及改造以后的效果。点击[下载地址](https://github.com/huaweicse/cse-java-chassis-samples/tree/master/springcloud-sample-cse-rpc)获取改造后的项目。

## 集成方式

CSE支持如下几种集成方式，当需要和Spring Cloud集成的时候，CSE可以作为一个Servlet替换org.springframework.web.servlet.DispatcherServlet。

![](/assets/open-design-integrate-with-running-environment.png)

## 改造步骤

通过依赖spring-boot-starter-transport，可以引入对于CSE的依赖。为了简单的接入CSE，还引入了cse-solution-service-engine。

```
<dependency>
  <groupId>org.apache.servicecomb</groupId>
  <artifactId>spring-boot-starter-transport</artifactId>
</dependency>
<dependency>
   <groupId>com.huawei.paas.cse</groupId>
   <artifactId>cse-solution-service-engine</artifactId>
</dependency>
```

## 配置启动类和定义REST接口

在启动类Application里面加入：@EnableServiceComb。这样就会在启动的时候，加载CSE运行时。

然后开发者就可以定义自己的REST接口（对应于Spring Cloud的Controller）。可以看出和Spring Cloud Controller的差异：

* 使用@RestSchema声明接口，并且指定Schema ID。CSE会对每个REST接口都生成一个接口定义文件，并上传到服务中心。Schema ID在微服务内部需要保持唯一。

* 显示的使用@RequestMapping定义路径。CSE支持JaxRS和SpringMVC两种方式定义REST接口，运行时根据这个标签来区分采用哪种方式生成契约。

其他服务的定义方式和Spring Cloud保持完全一致。CSE支持客户端以RestTemplate和RPC两种方式访问服务端，也可以通过浏览器使用REST的方式直接访问服务端，所以一个好的开发实践是给每个REST服务都定义一个接口Hello。使用CSE，不需要Spring Cloud的声明式REST调用（Feign），可以大大简化开发者的工作量。

```
@RestSchema(schemaId="hello")
@RequestMapping(path = "/hello", produces = MediaType.TEXT_PLAIN)
public class HelloService implements Hello {
  private static org.slf4j.Logger log = LoggerFactory.getLogger(HelloService.class);

  @Override
  @RequestMapping(path = "/sayhi", method = RequestMethod.GET)
  public String sayHi(@RequestParam(name = "name", required = false) String name) {
    log.info("Access /hello/sayhi, and name is " + name);
    return "from provider: Hello " + name;
  }
}
```

引入CSE后，不再需要Feign等组件，如下的一些依赖也可以移除。还有标签@EnableDiscoveryClient、@EnableZuulServer等
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```
## 定义微服务信息
在application.yml中配置基本的微服务信息。
```
# 应用名称
APPLICATION_ID: spring-cloud-application-cse-rpc
# 微服务名称和版本号，微服务名称使用Spring Cloud的名称
service_description:
  name: ${spring.application.name}
  version: 1.0.0
# 服务中心和配置中心的地址
servicecomb:
  service:
    registry:
      address: https://cse.cn-north-1.myhwclouds.com
      instance:
        watch: false
  config:
    client:
      serverUri: https://cse.cn-north-1.myhwclouds.com
      refreshMode: 1
      refresh_interval: 15000
# 对外发布的地址，端口号使用server.port
  rest:
    address: 0.0.0.0:${server.port}
# AK/SK认证信息，需要修改为自己的AK/SK
  credentials:
    accessKey: your access key
    secretKey: your secret key
    akskCustomCipher: default
# 线下开发，关闭监控数据上报
  monitor:
    client:
      enable: false
```

## 客户端访问

CSE简化了客户端访问服务端的方式，同时也支持Spring Cloud使用RestTemplate的方式去访问。

* RPC方式

```
  @RpcReference(microserviceName="helloprovider", schemaId="hello")
  Hello client;
  client.sayHi(name)
```

* RestTemplate方式

```
  RestTemplate restTemplate = RestTemplateBuilder.create();
  restTemplate.getForObject("cse://helloprovider/hello/sayHi?name=" + name, String.class);
```

## 体验改造后的服务
在构造的过程中，已经体验了开发上的便利：比Feign更好的RPC支持，以及在"快速接入"章节的相关功能。改造后的应用通过： http://localhost:7211/hello?name=3 进行访问。然后可以登录CSE，体验更多的治理功能。下面挑选了几个经常使用的功能进行描述。

### 服务契约
进入微服务引擎，微服务名录查看微服务信息，可以看到helloprovider包含如下接口定义文件。
```
swagger: "2.0"
info:
  version: "1.0.0"
  title: "swagger definition for io.provider.HelloService"
  x-java-interface: "cse.gen.spring_cloud_application_cse_rpc.helloprovider.hello.HelloServiceIntf"
basePath: "/hello"
consumes:
- "application/json"
produces:
- "application/json"
paths:
  /sayhi:
    get:
      operationId: "sayHi"
      parameters:
      - name: "name"
        in: "query"
        required: false
        type: "string"
      responses:
        200:
          description: "response of 200"
          schema:
            type: "string"
```
当需要使用浏览器、postman等HTTP客户端访问后台接口的时候，契约可以替代接口说明文档。

### 调用关系
在服务治理界面，通过图形化的方式展现了微服务之间的调用关系。
![](/assets/call-graph.png)

### 服务治理
在服务治理页面，对helloconsumer下发一个故障注入，接口调用模拟3秒的时延。

![](/assets/governance-example.png)

然后访问接口： http://localhost:7211/hello?name=3 可以发现这个接口返回时间被延长。

### 服务监控
将应用部署到华为云以后，微服务会统计和上报自己的监控状态，这样用户就可以通过仪表盘、ServiceStage的性能监控等功能，监控微服务的运行状态、调用链等指标。
注意：本地调试情况下，不会上报监控数据。并且日志可能打印如下错误：
```
Can not find any instances from service center due to previous errors. service=default/CseMonitoring/latest
```
如果不期望上报监控数据，可以增加配置项：
```
servicecomb.monitor.client.enable=false
```
关闭。
 