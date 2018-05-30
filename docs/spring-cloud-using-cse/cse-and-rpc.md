# 使用CSE作为RPC框架

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

在启动类Application里面加入@EnableServiceComb加载CSE运行时，并通过@SpringBootApplication(exclude=DispatcherServletAutoConfiguration.class)关闭Spring RESTful Web Service。

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
 
## 补充说明
除了上述可以直接感受到的功能，切换为CSE RPC后，请求处理流程也发生了变化。调用流程使用了CSE优秀的统一一致的处理流程。
![](/assets/open-design-running-arch.png)

该流程里面的处理链扩展能力和契约能力，是所有治理的基础。

当然，修改后，还会发生其他一些变化，业务代码还会涉及一些修改，这些修改包括REST接口定义的数据类型支持（参考[说明](../using-cse-in-spring-boot/diff-spring-mvc.md))，以及Spring Cloud其他的构建在Spring RESTful Web Service之上的能力。修改过程中，也可能会碰到若干jar包冲突或者不兼容的情况。

这些情况都不涉及到业务逻辑代码的修改，本质上只是改变了业务代码发布为服务的表现形式。使用CSE，能够更好的聚焦于业务逻辑开发。

## 改造过程中的常见问题

### HttpServletRequest
HttpServletRequest是J2EE(Servlet)协议定义的对象。CSE支持在Servlet协议上、HTTP协议以及其他协议上提供REST服务，因此不支持特定技术框架的对象。需要将接口定义修改为平台无关的原型。

以下面接口为例：

```
    @RequestMapping(value = "/auth", method = RequestMethod.POST)
    public ResultResponse createAuthenticationToken(HttpServletRequest request,
            @RequestBody JwtAuthenticationRequest authenticationRequest) throws AuthenticationException{
        String type = authenticationRequest.getType();
        String appCode = request.getHeader(BaseTypeConstants.HEADER_APP_CODE);
        String appType = request.getHeader(BaseTypeConstants.HEADER_APP_TYPE);
… …
```

修改后：

```
    @RequestMapping(value = "/auth", method = RequestMethod.POST)
public ResultResponse createAuthenticationToken(@RequestHeader(name= BaseTypeConstants.HEADER_APP_CODE) String appCode, 
@RequestHeader(name= BaseTypeConstants.HEADER_APP_TYPE String appType),
            @RequestBody JwtAuthenticationRequest authenticationRequest) throws AuthenticationException{
        String type = authenticationRequest.getType();

… …
```

对于HttpServletResponse的改造原理是一样的。

### Feign客户端
CSE提供了比Feign更加容易使用的客户端RPC方式，客户端不需要像Feign一样声明REST映射关系。本项目中已经包含了Feign的改造例子。

改造前：
```
# 客户端接口声明
@FeignClient("helloprovider")
@RequestMapping(path = "/hello")
public interface Hello {
  @RequestMapping(path = "/sayhi", method = RequestMethod.GET)
  String sayHi(@RequestParam(name = "name") String name);
}
# 客户端使用
@Autowired
Hello client;
```

改造后:
```
# 客户端接口声明
public interface Hello {
  String sayHi(String name);
}
# 客户端使用
@RpcReference(microserviceName="helloprovider", schemaId="hello")
Hello client;
```

对于习惯RPC编程的开发人员，可以直接在服务定义的时候，就声明接口，然后作为API发布。这样客户端就可以避免重复写代码了。

### 三方软件冲突

启动报如下错误：
```
Caused by: java.lang.IllegalStateException: Detected both log4j-over-slf4j.jar AND bound slf4j-log4j12.jar on the class path, preempting StackOverflowError.
```
spring boot默认使用logback， 并且依赖了slf4j-log4j12，会导致冲突。可以通过排除log4j-over-slf4j解除冲突:
```
<dependency>
    <groupId>com.huawei.paas.cse</groupId>
    <artifactId>cse-solution-service-engine</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

启动报如下错误：
```
java.lang.NoSuchMethodError: javax.ws.rs.core.Response$Status$Family.familyOf(I)Ljavax/ws/rs/core/Response$Status$Family;
    at org.apache.servicecomb.serviceregistry.client.http.ServiceRegistryClientImpl.registerSchema(ServiceRegistryClientImpl.java:309) ~[service-registry-1.0.0.B010.jar:1.0.0.B010]
```
这个属于jsr311-api冲突，如果pom中依赖了下面的旧的协议包，删除即可
```
<dependency>
    <groupId>javax.ws.rs</groupId>
    <artifactId>jsr311-api</artifactId>
    <version>1.1.1</version>
</dependency>
```


