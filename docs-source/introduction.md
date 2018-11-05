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
从华为云帮助中心可以获取[版本说明](https://support.huaweicloud.com/productdesc-cse/cse_productdesc_0008.html)。

