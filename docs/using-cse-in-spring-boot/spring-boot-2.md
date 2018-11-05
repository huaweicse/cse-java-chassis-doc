# 使用Spring Boot 2

***说明：需要使用CSE SDK 2.3.51及其以上版本***

针对Spring技术体系，CSE提供了如下几个组件，方便开发者。

* cse-dependency
这个依赖关系管理器提供CSE推荐的依赖关系管理。它的依赖关系会随着CSE版本的发展进行调整。比如在spring 4是稳定版本的时候，这个管理器提供spring 4的依赖；spring 5是稳定版本的时候，这个管理器提供spring 5的依赖。当前CSE提供的版本默认为spring 4. 

  * spring4升级到spring 5的注意事项
    * spring 5和spring 4在很多接口和使用方法上是不兼容的。因此CSE针对这些不兼容的实现提供了对应的处理措施。比如：EnableServiceComb：spring 4为org.apache.servicecomb.springboot.starter.provider.EnableServiceComb, spring 5为org.apache.servicecomb.springboot2.starter.EnableServiceComb; DispatcherServletAutoConfiguration: spring 4为org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration, spring5为org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration。
    * spring 5废弃了AsyncRestTemplate。CSE提供了AsyncRestTemplate的实现，因此也废弃这个使用方式。
    * 如果使用了bean配置文件，并且定义了如下schema，需要调整：xsi:schemaLocation="http://www.springframework.org/schema/beans classpath:org/springframework/beans/factory/xml/spring-beans-3.0.xsd” 修改为：xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd“

* cse-dependency-spring4

如果业务代码没使用spring 5或者spring boot 2，可以使用这个管理器。

* cse-dependency-spring5

如果业务代码使用spring 5，可以使用这个管理器。

* cse-dependency-spring-boot1

如果业务代码使用spring boot 1和spring 4，可以使用这个管理器。

* cse-dependency-spring-boot2

如果业务代码使用spring boot 2和spring 5，可以使用这个管理器。

## 如何使用

开发者可以利用maven提供的dependencyManagement更好的管理依赖关系。

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.huawei.paas.cse</groupId>
            <artifactId>cse-dependency</artifactId>
            <version>${paas.cse.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

* [CSE 支持spring 4/5 以及spring boot 1/2 maven组件依赖关系配置参考](https://bbs.huaweicloud.com/blogs/33eaad00db4911e8bd5a7ca23e93a891)提供了配置和原理说明。

* [基于Spring Boot 2.0的IoT应用集成和使用CSE实践](https://bbs.huaweicloud.com/blogs/02de5f11cb6e11e8bd5a7ca23e93a891)提供了一个实际Spring Boot 2的例子。
