# 连接服务中心

## 功能描述

服务中心实现注册和发现，在CSE/ServiceStage查看微服务目录，都需要微服务连接上服务中心。连接服务中心主要的步骤包括引入认证模块、配置认证信息。

## 配置参考

* 增加依赖关系\(pom.xml\)

相对于ServiceComb，CSE的服务中心增加了认证，需要在代码中引入认证模块。

```xml
<dependency> 
  <groupId>com.huawei.paas.cse</groupId>  
  <artifactId>foundation-auth</artifactId> 
</dependency>
```

* 配置项\(microservice.yaml）

配置项包括服务中心的地址和认证信息。

```xml
servicecomb:
 service:
  registry:
   address: https://cse.cn-north-1.myhwclouds.com    #根据实际地址配置服务中心地址
   instance:
     watch: false #使用API网关访问，只能使用PULL模式

  credentials:
    accessKey: your access key
    secretKey: your secret key
    akskCustomCipher: default #加密选项
    project：cn-north-1 #可选，所属project。
```

服务中心的的地址可以在CSE的工具下载页面查看，比如该页面有如下消息。根据租户所在的区域不同，地址会有差异。

```
服务中心地址、配置中心地址和仪表盘上报地址为： https://cse.cn-north-1.myhuaweicloud.com:443
```

认证信息可以从公有云凭证管理获取，加密选项的使用提供了安全存储AK/SK的机制，参考[说明](../tools/ak-sk-encrypt.md)