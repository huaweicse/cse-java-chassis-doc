# 连接配置中心

## 功能描述

配置中心实现配置下发，连接配置中心是使用治理、灰度发布等功能的前提。CSE对于用户微服务的治理能力，都是构建在动态配置之上的。

## 配置参考

使用配置中心的步骤和服务中心一样，增加模块依赖和认证信息。这里省略了认证信息配置的内容。

* 增加依赖关系\(pom.xml\)

```xml
<dependency> 
  <groupId>org.apache.servicecomb</groupId>
  <artifactId>config-cc</artifactId>
</dependency>
```

* 启用配置\(microservice.yaml）

```yaml
cse:
 config:
  client:
   serverUri: https://cse.cn-north-1.myhwclouds.com
   refreshMode: 1 #使用API网关访问，只能使用PULL模式
```
