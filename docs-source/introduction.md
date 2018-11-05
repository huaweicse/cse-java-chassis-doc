# 概述

CSE\(Cloud Service Engine\) Java SDK是华为推出的产品级微服务开发框架，已经在华为内部多个大型产品上得到了使用和验证。使用CSE Java SDK开发微服务，可以最大化的简化开发门槛，提升产品上线速度。同时可以获得微服务运行时高可靠性保证、运行时动态治理等一系列开箱即用的能力。

CSE Java SDK 100% 兼容 [ServiceComb Java Chassis](https://github.com/apache/incubator-servicecomb-java-chassis)，开发者可以参考[ServiceComb Java Chassis开发指南](https://docs.servicecomb.io/java-chassis/zh_CN/)获取开发指导。本文主要描述CSE Java SDK的扩展和易用性增强内容，以及与其他开发框架集成，在华为云上部署等主题。

为了描述简单，本文会使用CSE指代CSE Java SDK，使用ServiceComb指代ServiceComb Java Chassis。

开发者可以通过[微服务引擎华为云官网](https://www.huaweicloud.com/product/cse.html)了解CSE。如有疑问，可通过[CSE论坛](http://forum.huaweicloud.com/forum.php?mod=forumdisplay&fid=622)进行咨询。

## 版本获取

### 通过华为开源镜像站下载setting.xml文件

访问[华为开源镜像站](https://mirrors.huaweicloud.com/)，搜索“HuaweiCloud”，点击“HuaweiCloud SDK”后下载setting.xml。

### 手工配置setting.xml文件

依次按照下面方法手动修改settings.xml文件

1、在profiles节点中添加如下内容：
```
<profile>
    <id>MyProfile</id>
    <repositories>
        <repository>
            <id>HuaweiCloudSDK</id>
            <url>https://repo.huaweicloud.com/repository/maven/huaweicloudsdk/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</profile>
```
2、在mirrors节点中增加：
```
<mirror>
    <id>huaweicloud</id>
    <mirrorOf>*,!HuaweiCloudSDK</mirrorOf>
    <url>https://repo.huaweicloud.com/repository/maven/</url>
</mirror>
```

3、增加activeProfiles标签激活配置：
```
<activeProfiles>
    <activeProfile>MyProfile</activeProfile>
</activeProfiles>
```

## 版本说明
### 2.3.35
#### 新特性
* [SCB-728] 从Edge进来的请求，支持form格式能力增强。允许key值为json的转换。
* [SCB-708] Spring MVC编码方式支持query参数为一个POJO对象，简化接口参数个数。
* [SCB-729] 提供一种机制，可以由运维工具诊断、查询和更新客户端缓存的实例列表。
* [SCB-775] RestTemplate支持使用弱类型，比如JsonObject或者String，来访问服务端接口。
* [SCB-778] Servlet模式下，支持注册容器路径到swagger的base path，从而支持RestTemplate以URL的全路径访问服务端。
* [SCB-777] JAX-RS开发模式支持BeanParam
* [SCB-687] Highway支持通过servicecomb.highway.server.connection-limit设置最大连接数

#### 修改特性
* [SCB-775] RestObjectMapper.INSTANCE修改为RestObjectMapperFactory.getRestObjectMapper()
* [CSE-2201] cse.credentials.*,cse.monitor.*等配置项支持别名servicecomb.credentials.*,servicecomb.monitor.*
* [CSE-2256] servicecomb.monitor.client.sslEnalbed修改为servicecomb.monitor.client.sslEnabled
* [CSE-2256] servicecomb.monitor.client.enable修改为servicecomb.monitor.client.enabled
* [SCB-795] jackson 从 2.9.5 升级到 2.9.6

#### Bug fixes
* [CSE-2202]cse-solution-service-engine携带了edge模块，会改变业务代码执行线程池，变成reactive模式，移除这个依赖。
* [DTS2018071604849]监控服务的地址配置，不支持URL不书写端口号并使用缺省端口号的问题。(http:80, https:443)。
* [SCB-754] 解决业务扩展HttpServerFilter抛出异常（NPE）的情况下，服务端没有正确写响应导致请求超时的问题。
* [SCB-762] linux环境扫描main class路径错误导致的spring加载路径找不到问题以及生成log4j合并文件报错的问题。
* [SCB-755] 解决打印大量“cse.*和servicecomb.*配置项重复”日志的问题
* [SCB-769] 解决启用故障注入时，可能导致客户端无法取到服务响应的问题。
* [SCB-774] 解决reactive模式下，优雅停机时打印大量警告日志问题。
* [SCB-780] 解决会话保持模式下，实例移除时，会话周期内继续访问移除的实例的问题。
* [SCB-787] 实例下线后，客户端会很长时间继续Ping实例状态
* [SCB-794] 从Edge调用Tomcat作为容器的服务，如果Filter返回401错误码，浏览器获取到错误码是490的问题。

### 2.3.30
#### 新特性
* [SCB-203] J2EE(Servlet）运行环境，支持文件上传和下载。
* [SCB-627] 客户端请求超时时间，支持按照接口级别进行动态设置。
* [SCB-636] 配置映射规则优化，支持从不同模块的映射文件加载映射规则，支持一个Key映射到多个Key。
* [CSE-2058]公有云支持使用PAAS_CSE_ENDPOINT环境变量指定服务中心、配置中心地址
* [SCB-625] 支持通过SPI的方式扩展Produces类型
* [SCB-697][SCB-685] JAX-RS, Spring MVC开发模式，支持使用标签定义参数缺失值
* [SCB-679] 支持跨域请求访问设置
* [SCB-506] 服务治理相关的事件（熔断、实例隔离）发生的时候，发送event，支持业务开发事件上报功能
* [SCB-640] 基于publicKey提供黑白名单功能。
* [SCB-725] Spring启动方式的时候，默认扫描main所在Class的package，简化用户配置。
* [SCB-733] 开放LocalContext接口供开发者使用
* [SCB-726] 从Edge进来的请求，支持form格式，Edge自动将form转换为json。
* [SCB-706] 提供客户端ping机制，能够通过ping扩展，检测客户端缓存实例是否可用。该功能默认启用，配合实例隔离功能对检测失败的实例进行隔离。

#### 修改特性
* [SCB-546] 微服务契约注册优化，当检测到相同微服务版本契约不同，并且环境不是development的时候，启动失败。环境配置项调整为service_description.environment
* [SCB-679] cse.executor.groupThreadPool，cse.executor.reactive增加别名servicecomb.executor.groupThreadPoo，servicecomb.executor.reactive。
* [SCB-194] BeanUtils.init 缺省扫描main函数所在的package，简化用户配置。
* [SCB-712] 缺省情况下，不往服务中心注册通过schema定义解析出来的path信息，需要通过配置项servicecomb.service.registry.registerPath=true开启。
* [SCB-671] 为了兼容，早期将servicecomb.xx配置项复制为cse.xxx，修改为将cse.xxx配置项复制为servicecomb.xx配置项。开发者在程序中、配置文件优先使用servicecomb.xx。
* [SCB-671] 配置文件优先级配置项cse-config-order调整为servicecomb-config-order
* [SCB-671] 配置项cse.configurationSource.additionalUrls和cse.configurationSource.defaultFileName分别调整为servicecomb.configurationSource.defaultFileName和 servicecomb.configurationSource.defaultFileName
* [SCB-727] 检测到服务中心和本地的schema不相同的时候，打印schema内容到日志，降低定位成本。
* [SCB-706] 将实例隔离、基于属性路由、数据中心路由等功能切换为DiscoveryFilter。详细开发文档参考：https://huaweicse.github.io/servicecomb-java-chassis-doc/zh_CN/references-handlers/loadbalance.html
* [SCB-706] 调整ServerListFilterExt的实现，不再继承Ribbon的ServerListFilter，并且提供了新方法：public List<Server> getFilteredListOfServers(List<Server> servers, Invocation invocation)以支持基于Invocation属性的过滤器。开发者如果使用了ServerListFilterExt扩展，会编译不通过，可以参考文档https://huaweicse.github.io/servicecomb-java-chassis-doc/zh_CN/references-handlers/loadbalance.html查看是否需要进行自定义扩展。如果需要，可以通过扩展新的ServerListFilterExt或者DiscoveryTree来实现。
* [SCB-706] CseServer重命名，调整为ServiceCombServer，并去掉了LoadBalancerStats属性。
* [SCB-706] 默认开启实例隔离能力；如果依赖了cse-solution-service-engine，还将默认开启重试。

#### Bug fixes
* [SCB-653]解决Edge在转发Tomcat容器提供REST服务的请求的时候，如果采用Trunked编码，浏览器（或者三方HTTP工具）解析响应失败的问题。
* [SCB-646]解决Jax RS开发模式下，手工写契约，仍然自动生成契约并报错的问题。
* [SCB-658]解决Edge Service MicroserviceVersion可能存在的内存泄露问题。
* [SCB-656]解决Edge Service异常情况下没有返回Provider的错误码，而是返回502的问题。
* [SCB-654]解决DiscoveryTree并发访问问题。
* [Issue #573]灰度发布使用大小写敏感的规则的时候，如果参数值为空，会执行异常
* [SCB-669] 解决Generics参数类型里面还包含Generics字段的情况下，响应解析失败的问题
* [SCB-667] 解决在异常场景下,无法优雅退出的问题
* [SCB-662]解决cse.*和servicecomb.*配置项并存的情况下，读取参数值错误的问题
* [SCB-651] 解决流控第一个周期的数值和后面周期的数值差1的问题
* [SCB-703] 解决返回值为void时，反序列化失败的问题
* [SCB-715] 解决ContextClassLoader为空时，抛出NullPointerException的问题
* [SCB-701] 解决RequestBody(required = false)场景下，解析空body抛出异常的问题
* [SCB-705] 解决服务端未启动，先启动客户端并调用接口，服务端启动后仍然无法正常调用客户端的问题。
* [SCB-693] 解决启动过程中获取网卡信息失败，可能导致的服务注册失败问题
* [SCB-696] 解决Access Log在短连接情况下没有正确打印HOST信息的问题
* [SCB-738] 解决服务删除的场景下，客户端会继续周期查询服务版本的问题。

### 2.3.25
#### 新特性
* [SCB-617] J2EE运行环境（tomcat）优雅退出支持；Vert.x运行环境优雅退出增加了事件机制，并且统一了两个环境优雅退出的逻辑。
* [SCB-611] Edge Service提供了两种通用的路由管理机制：基于服务名和版本的灰度管理和基于URL映射配置的灰度管理机制，简化用户使用Edge Service。
* [SCB-607] Access Log支持打印Context头信息；格式支持用户扩展。

#### 修改特性
* [SCB-589] 采用Edge一样的机制管理Consumer的契约，默认拉取0+所有版本信息（之前是拉起latest），并对不同的版本采用隔离的ClassLoader进行接口调用，以解决Provider接口变更，Consumer配套Provider修改，升级的时候，Consumer先升级导致的接口调用异常和无法恢复问题。
* [SCB-590] validation-api由1.1.0升级到2.0.0
* [SCB-590] hibernate-validator由5.2.4.Final升级到6.0.2.Final。
* [SCB-590] hibernate-validator-annotation-processor由5.2.2.Final升级到6.0.2.Final。

#### Bug fixes
* [SCB-471] 解决配置中心在watch模式下循环打印NullPointerException的问题

### 2.3.23
#### 新特性
* [SCB-532] 接口定义支持自引用的数据类型
* [SCB-548] 优雅停机支持，服务停止的时候，正常关闭网络线程、等待调用完成等
* [SCB-482] SDK支持http2作为通信协议
* [SCB-531] 契约定义x-java-interface支持可选

#### 修改特性
* 匹配ServiceComb Java Chassis的版本：commit id 5fc99f5
* [SCB-546] 微服务契约注册优化，当检测到相同微服务版本契约不同，并且环境不是development的时候，启动失败。
* [SCB-582] 提供服务中心网络暂时不可达、服务中心故障重启等异常情况下的实例保护机制。当从服务中心查询到空实例的时候，默认会对现在使用的实例Ping一次，Ping成功则不移除本地实例。查询的实例列表不为空的时候，行为保持不变。
* [SCB-504] Spring 版本由4.3.5.RELEASE升级到4.3.16.RELEASE。注意：新版本Spring的原生RestTempalte接口在未知HTTP错误码的情况下，接口行为有变化。使用CSE提供的接口则不受影响。
* [SCB-542] Netty 版本有4.1.17.Final升级到4.1.24.Final

#### Bug fixes
* [SCB-591] 解决查询配置中心配置项是，微服务名称没有转码导致查询失败的问题
* [SCB-580] 解决大文件上传抛出异常的问题
* [SCB-579] 解决Consumer上传空文件，服务端抛出NPE的问题
* [SCB-607] 解决服务中心没启动情况下，先启动微服务，再启动服务中心，仍然无法注册的问题
* [SCB-543] 解决注册微服务过程中删除微服务信息，后续微服务无法继续注册的问题
