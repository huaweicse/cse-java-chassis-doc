# 微服务接口兼容常见问题

在进行微服务持续迭代开发的过程中，由于新特性在不停的加入，一些过时的特性在不停的修改，接口兼容问题面临巨大的挑战，特别是在运行环境多版本共存（灰度发布）的情况下。本章节主要描述接口兼容管理的一些实践建议，以及在使用CSE过程中碰到了兼容性问题的解决办法。由于微服务一般都通过REST接口对外提供服务，没有特殊说明的情况下，这里的接口都指REST接口。

## 保证接口兼容的实践

为了防止接口兼容问题，开发者在进行接口变更（新增、修改、删除等）的时候，建议遵循下面的一些原则。

1. 只增加接口，不修改、不删除接口。
2. 作为Provider，增加接口的时候，相应的将微服务版本号递增。比如将2.1.2修改为2.1.3。版本号按照规范使用x.y.z的格式，只包含数字便于管理，建议每位数字不大于125。
3. 作为Consumer，使用Provider的新接口时候，指定Provider的最小版本号。比如：cse.references.\[serviceName\].version-rule=2.1.3+，其中serviceName为Provider的微服务名称。
4. 在服务中心，定期清理不再使用的老版本的微服务信息。

## 接口兼容常见问题及其解决办法

1. 开发阶段，由于存在频繁的接口修改，又不想频繁修改版本号，容易本地和服务中心契约不一致，且契约未被允许更新到服务中心，导致调试的时候接口调用失败的情况。

推荐使用CSE提供了微服务按environment区分、隔离的能力(当前支持development和production)，允许处于development环境的微服务在不升级版本的情况下，仅需重启服务即可重新注册契约到服务中心。

所有微服务在microservice.yaml中增如下配置，且需要在Provider启动后，再重启Consumer(若请求走edge，需要重启edge服务)：

```
service_description:
  name: xxx-service
  version: 0.0.1
  environment: development
```

2. 开发阶段，由于存在频繁的接口修改，也不会清理服务中心的数据，容易出现调试的时候接口调用失败的情况。

推荐使用华为公有云在线的服务中心，可以直接登录使用微服务引擎提供的微服务管理功能删除微服务或微服务实例。

微服务引擎也提供了[本地轻量化服务中心](https://console.huaweicloud.com/cse/?region=cn-north-1#/cse/tools)，将服务停止后即可清理服务中心数据。服务中心及其frontend代码已开源，[项目地址](https://github.com/apache/incubator-servicecomb-service-center)。

3. 发布阶段，需要审视下接口兼容的实践的步骤，确保不在线上引入接口兼容问题。

如果不小心漏了其中的某个步骤，则可能导致如下一些接口兼容问题：

* 如果修改、删除接口：导致一些老的Consumer将请求路由到新的Provider，调用失败。

解决办法：指定Provider的版本号、或修改Consumer适配新的Provider。

* 如果忘记修改微服务版本号：导致一些新的Consumer将请求路由到老的Provider，调用失败。

解决办法：升级Provider版本号、删除老的Provider实例、重启Consumer。

* 如果忘记配置Consumer的最小依赖版本：当部署顺序为先停止Consumer，再启动Consumer，再停止Provider，再启动Provider的情况，Consumer无法获取到新接口信息，就采用了老接口，当Provider启动以后，Consumer发起对新接口的调用会失败；或者在Provider没启动前，调用新接口失败等。

解决办法：建议先启动Provider，再启动Consumer。

通用规避措施：出现的接口兼容问题不同，处理方式会有差异。极端情况，只需要清理Provider、Consumer的微服务信息，然后重启微服务即可。当服务调用关系复杂的情况下，接口兼容问题影响范围会更加广泛，同时清理Provider、Consumer数据会变得复杂，因此建议遵循上面的规范，避免不兼容的情况发生。



## 常见的接口不兼容情况的日志

* consumer method \[com.huawei.paas.cse.demo.CodeFirstPojoIntf:testUserMap\] not exist in swagger

可能是Provider增加了接口，但是没有更新版本号。需要删除微服务数据或者更新版本号后重新启动Provider，并重启Consumer。



* 契约或接口变更(含增删查改、参数变化等)，但environment未设定为development，契约不允许更新。schemaId为download、upload的两个契约已存在，但新增的schemaId为TaskTemplateController的无法注册，相应接口自然会调用失败。需要升级版本号，或指定environment为development。

```
2018-06-14 22:51:55,239 [ERROR] SchemaIds is different between local and service center. Please change microservice version. id=1f4c94c66fe011e8945700ff37174dd4 appId=uploadapp, name=upload-service, version=0.0.1, local schemaIds=[download, upload, TaskTemplateController], service center schemaIds=[download, upload] org.apache.servicecomb.serviceregistry.task.MicroserviceRegisterTask.checkSchemaIdSet(MicroserviceRegisterTask.java:116)
2018-06-14 22:51:55,243 [INFO] schemaId download exists true org.apache.servicecomb.serviceregistry.task.MicroserviceRegisterTask.registerSchemas(MicroserviceRegisterTask.java:144)
2018-06-14 22:51:55,246 [INFO] schemaId upload exists true org.apache.servicecomb.serviceregistry.task.MicroserviceRegisterTask.registerSchemas(MicroserviceRegisterTask.java:144)
2018-06-14 22:51:55,249 [WARN] get response for org.apache.servicecomb.serviceregistry.api.response.GetExistenceResponse failed, 400:Bad Request, {"errorCode":"400016","errorMessage":"Schema does not exist","detail":"schema does not exist."}
 org.apache.servicecomb.serviceregistry.client.http.ServiceRegistryClientImpl.lambda$null$0(ServiceRegistryClientImpl.java:118)
2018-06-14 22:51:55,250 [INFO] schemaId TaskTemplateController exists false org.apache.servicecomb.serviceregistry.task.MicroserviceRegisterTask.registerSchemas(MicroserviceRegisterTask.java:144)
2018-06-14 22:51:55,258 [ERROR] Register schema 1f4c94c66fe011e8945700ff37174dd4/TaskTemplateController failed, statusCode: 400, statusMessage: Bad Request, description: {"errorCode":"400014","errorMessage":"Undefined schema id","detail":"schemaId non-exist， can't be added, environment is production"}
. org.apache.servicecomb.serviceregistry.client.http.ServiceRegistryClientImpl.registerSchema(ServiceRegistryClientImpl.java:306)
```

* 

```
2018-06-14 22:23:59,407 [WARN] {"errorCode":"400012","errorMessage":"Micro-service does not exist","detail":"provider not exist, consumer 8e24bc416fde11e8945700ff37174dd4 find provider default/CseMonitoring/latest"}
 org.apache.servicecomb.serviceregistry.client.http.ServiceRegistryClientImpl.lambda$null$4(ServiceRegistryClientImpl.java:199)
2018-06-14 22:23:59,408 [ERROR] Can not find any instances from service center due to previous errors. service=default/CseMonitoring/latest org.apache.servicecomb.serviceregistry.registry.AbstractServiceRegistry.findServiceInstances(AbstractServiceRegistry.java:256)
```


