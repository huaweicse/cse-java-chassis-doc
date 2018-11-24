### 简介

&emsp;&emsp;服务契约，指基于OpenAPI规范的微服务接口契约，是服务端与消费端对于接口的定义。服务契约用于服务端和消费端的解耦，服务端围绕契约进行服务的实现，消费端根据契约进行服务的调用。CSE Java SDK使用yaml文件格式定义服务契约，可支持多种风格开发微服务。

&emsp;&emsp;CSE-Codegen是基于Swagger Codegen实现的代码生成工具，用户只需在微服务工程的服务端和消费端的pom文件分别引入插件依赖，就可以根据定义好的契约文件生成服务端和消费端的框架代码，快速构建微服务应用。

### 特性描述

1. 新增契约同步功能

  支持从远端的Git仓库同步一个或多个契约到微服务工程中，每次运行都可以根据最新的契约生成框架代码。

2. 服务提供者推荐使用SpringMVC风格，服务消费者推荐使用透明RPC风格

  在不指明language的情况下，为服务提供者生成SpringMVC风格的框架代码，为服务消费者生成透明RPC风格的框架代码。
3. 为服务提供者和服务消费者生成完备的框架性代码

  通过在微服务工程的pom文件中配置插件依赖，运行插件后生成以下文件：服务提供者provider生成model + delegate + controller + impl，服务消费者consumer生成model + delegate + impl。

4. 适应多服务多契约的场景

  在配置中增加参数，可以适应多服务多契约的场景。

5. 使用契约中的x-java-class参数，避免consumer和provider的model路径不一致导致调用失败

  x-java-class作为契约中一个重要的参数，存在于definitions中的每一个model，标志着model的package路径，能够保证服务消费者consumer和服务提供者provider的model的package路径统一。要求契约的每个model都具备x-java-class，根据x-java-class > service.packageName > packageName的优先级生成model的pacakge路径，避免consumer和provider的model路径不一致导致调用失败。

6. 最大程度保证显式契约和隐式契约的一致性

  契约必须按照标准的Swagger API Spec语法来描述，使用yaml来表示。  
  契约中建议model都使用x-java-class参数，保证package路径统一。否则，使用REST方式会调用失败，使用RPC方式会使性能下降。  
  契约中不建议使用default返回码，CSE Java SDK不支持default返回码，插件也默认屏蔽default返回码。  
  插件严格按照契约定义，针对契约的不同返回码，生成的框架代码有所体现，最大程度保证显式契约和隐式契约一致。


### 版本
&emsp;&emsp;目前CSE-Codegen插件最新版本是2.2.8，可以到华为镜像站的[huaweicloudsdk仓库](https://repo.huaweicloud.com/repository/maven/huaweicloudsdk/)获取。
