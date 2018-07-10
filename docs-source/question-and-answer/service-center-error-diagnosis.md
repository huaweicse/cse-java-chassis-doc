# 连接华为公有云服务中心常见问题

开发者可以直接使用华为公有云提供的服务中心进行开发工作。使用服务中心，开发者需要注册华为云账号，并在"我的凭据”里面生成AK/SK信息用于访问认证控制。如何接入华为云的详细信息请参考“[华为公有云上部署](/start/deployment-on-cloud.md)”。

本章节重点介绍连接服务中心一些常见的异常及其排查方法。

## \#1

* 异常消息

{"errorCode":"401002","errorMessage":"Request unauthorized","detail":"Invalid request, header is invalid, ak sk or project is empty."}

* 问题原因

AK、SK没有正确设置和携带到请求头里面。

* 排查方法

检查项目中是否依赖如下认证模块。（间接依赖也是可以的，比如依赖cse-solution-service-engine\)

```
<groupId>com.huawei.paas.cse</groupId>
<artifactId>foundation-auth</artifactId>
```

检查microservice.yaml中的ak/sk配置是否正确，accessKey和secretKey是否填写错误，一般secretKey长度比accessKey长。

```
cse:
  credentials:
    accessKey: your access key
    secretKey: your serect key
    akskCustomCipher: default
```

可以登陆华为云，在“我的凭证”里面查询到accessKey信息，secretKey由用户自己保存，无法查询。如果忘记相关凭证，可以删除凭证信息，生成新的凭证信息。

## \#2

* 异常消息

{"errorCode":"401002","errorMessage":"Request unauthorized","detail":"Get service token from iam proxy failed,{\"error\":\"validate ak sk error\"}"}

* 问题原因

AK、SK不正确。

* 排查方法

检查microservice.yaml中的ak/sk配置是否正确。可以登陆华为云，在“我的凭证”里面查询到accessKey信息，secretKey由用户自己保存，无法查询。如果忘记相关凭证，可以删除凭证信息，生成新的凭证信息。

## \#3

* 异常消息

{"errorCode":"401002","errorMessage":"Request unauthorized","detail":"Get service token from iam proxy failed,{\"error\":\"get project token from iam failed. error:http post failed, statuscode: 400\"}"}

* 问题原因

Project名称不正确。

* 排查方法

检查配置项cse.credentials.project的值是否正确，在“我的凭证”里面查询正确的Project名称。如果没有这个配置项，默认会根据服务中心的域名进行判断。当域名也不包含合法的Project名称的时候，需要增加这个配置项，保证其名称是“我的凭证”里面合法的Project名称。

## \#4

* 异常消息

{"errorCode":"400001","errorMessage":"Invalid parameter\(s\)","detail":"Version validate failed, rule: {Length: 64,Length: ^\[a-zA-Z0-9\_

[\\-.\]\*$}](\\-.]*$})

"}

* 问题原因

使用新版本SDK连接服务中心的老版本。

* 排查方法

检查服务中心的版本。可以从华为云官网下载最新版本的服务中心，或者从ServiceComb官网下载最新版本的服务中心。

## \#5

* 异常消息

Error message is [failed to resolve 'cse.cn-north-1.myhuaweicloud.com'. Exceeded max queries per resolve 4 ].

* 问题原因

CSE域名（cse.cn-north-1.myhuaweicloud.com）解析失败。

* 排查方法

请在microservice.yaml中将addressResolver.servers配置为电信的DNS Sever，如114.114.114.114或者114.114.115.115，或者8.8.8.8

addressResolver:
  servers: 114.114.114.114



## \#6

- 异常消息

{"errorCode":"400100","errorMessage":"Not enough quota","detail":"no quota to create instance, ..."}

- 问题原因

没有足够的额度增加服务实例。

- 排查方法

登录华为云，在微服务引擎页面，可以看到实例个数的额度。如果发现页面有额度，需要检查下代码配置的服务中心地址和区域信息，如果是华北区，则检查华北区的额度。