# 概述

CSE\(Cloud Service Engine\) Java SDK是华为推出的产品级微服务开发框架，已经在华为内部多个大型产品上得到了使用和验证。使用CSE Java SDK开发微服务，可以最大化的简化开发门槛，提升产品上线速度。同时可以获得微服务运行时高可靠性保证、运行时动态治理等一系列开箱即用的能力。

CSE Java SDK 100% 兼容 [ServiceComb Java Chassis](https://github.com/apache/incubator-servicecomb-java-chassis)，开发者可以参考[ServiceComb Java Chassis开发指南](https://huaweicse.github.io/servicecomb-java-chassis-doc/zh_CN/)获取开发指导。本文主要描述CSE Java SDK的扩展和易用性增强内容，以及与其他开发框架集成，在华为云上部署等主题。

为了描述简单，本文会使用CSE指代CSE Java SDK，使用ServiceComb指代ServiceComb Java Chassis。

开发者可以通过[微服务引擎华为云官网](https://www.huaweicloud.com/product/cse.html)了解CSE。如有疑问，可通过[CSE论坛](http://forum.huaweicloud.com/forum.php?mod=forumdisplay&fid=622)进行咨询。

## 版本获取

获取CSE需要使用如下maven仓库
```
<mirror>
  <id>mirrorId</id>
  <mirrorOf>central</mirrorOf>
  <name>Mirror of central repository.</name>
  <url>http://maven.huaweicse.com/nexus/content/groups/public</url>
</mirror>
```

## 版本说明

### CSE 2.3.25
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

### CSE 2.3.23
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