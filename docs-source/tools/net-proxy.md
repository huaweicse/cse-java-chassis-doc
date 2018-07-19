# 网络代理

## 场景描述

在微服务的开发过程中，有些开发人员因为生活或者工作的原因，会需要通过代理服务器来访问华为云。

## 代理配置

```yaml
cse:
  proxy:
    enable: true
    host: your proxy server
    port: your proxy server port
    username: user name
    passwd: password for proxy
```

> 代理配置很容易理解，分别配置 代理服务器，端口，用户名和密码。

# 服务监控

## 应用场景

将应用部署到华为云以后，微服务会统计和上报自己的监控状态，这样用户就可以通过仪表盘、ServiceStage的性能监控等功能，监控微服务的运行状态、调用链等指标。

## 监控配置

```yaml
cse:
  config:
    tenantName: default # 租户名称
    domainName: default #缺省值为default。作为微服务提供者，用于表明自身所属租户信息。微服务在发现实例的时候，只能被相同租户下的消费者发现。

  monitor:
    api:
      version: v2 # 当前项目接口的版本号，默认是 v2
    serverUri: #服务器的地址
    sslEnalbed: true # request 请求是否需要 TLS 验证，默认是 true
    enable: true   #是否 上报监控数据，默认是 true
    timeout: 5000 #与服务器建立连接时，请求超时时长，默认是 5000
    interval: 10000 #定时上传监控数据的时间间隔 ，默认是 10000
```

# 灰度发布

## 应用场景

灰度发布用于引流。用户可以根据比例、条件将服务请求转发到符合要求的实例上去。比如在升级的场景，将部分请求引流到新版本，部分请求引流到老版本。灰度发布在loadbalance提供，同时需要扩展几个filter。

## 配置管理

在 Consumer 端配置负载均衡 (microservice.yaml)，启用灰度发布的 filter
```yaml
cse:
 loadbalance:
  serverListFilters: darklaunch
  serverListFilter:
    darklaunch:
      className: com.huawei.paas.darklaunch.DarklaunchServerListFilter
```
## 配置灰度发布 规则

用户可以根据自己的需求在配置文件里面配置自己的灰度发布规则。
```yaml
#灰度发布规则的配置项如下：
{
    "policyType": "RULE", #这是一个枚举类型，定义了 "RULE" 和 "RATE" 两种，这个配置项决定了 如何对 ruleItems 中的规则进行解析
    "ruleItems": [
        {
            "groupName": "test",  #分组的名称
            "groupCondition": "version>0.0.1&&version<=0.0.4", #根据微服务的 version进行筛选
            "policyCondition": "city=chengdu",  # 根据 微服务中具体的属性进行筛选
            "caseInsensitive": false # 大小写是否敏感
        }，
        {

        }
    ]
}


{
    "policyType": "RATE",
    "ruleItems": [
        {
            "groupName": "test",  #分组的名称
            "groupCondition": "version>0.0.1&&version<=0.0.4", #根据微服务的 version进行筛选
            "policyCondition": "20",  # 单个小组可能被选中的百分比
            "caseInsensitive": false # 大小写是否敏感
        }
    ]
}

```
> 1. 因为在根据`version`进行分组的时候，会把所有的server分为 两大类，`第一大类`是符合其中某组的筛选条件，进入其中的某个小组的server。`第二大类`是不符合所有分组的筛选条件，进入defaultGroup 中的server。而当 `policyType` 为 `"RATE"` 的时候， 从第一大类随机挑出一个小组的百分比可能性为 `policyCondition * ruleItems.size` 。 例如上例，说明从`第一大类`随机挑出一个小组的可能性为 `20% * 1 `。 当 `policyType` 为 `"RULE"` 的时候，是从`第一大类`中挑选出第一个符合 `groupCondition` 和 `policyCondition` 的小组。如果没有小组符合，就返回`第二大类`中的server。

>2. 要想配置灰度发布的过滤规则，需要在配置文件中加入键值对
`cse.darklaunch.policy.%s ：%jsonString` ,`%s` 指的是你的 `微服务名称` ，`%jsonString` 指的是 `上述json配置项的字符串形式`
