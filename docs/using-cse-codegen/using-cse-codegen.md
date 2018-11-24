### 使用CSE-Codegen插件快速开发微服务

使用CSE-Codegen插件开发微服务步骤很简单，只需要准备一个Git仓库用于存放契约，在微服务工程里面引入插件依赖，运行插件就能同步远端契约到本地并根据契约生成框架代码。可以参考[项目示例代码](https://github.com/huaweicse/cse-java-chassis-samples/tree/master/cse-codegen-demo)，步骤如下。目前CSE-Codegen的版本已经更新到2.2.8并上传到华为镜像站的[huaweicloudsdk仓库](https://repo.huaweicloud.com/repository/maven/huaweicloudsdk/)，后续版本也会发布到这里。

###### 1. Git仓库归档契约
创建远端Git仓库，用于契约管控，上传契约文件到Git仓库中。可以参考[契约仓库示例](https://github.com/huaweicse/cse-codegen-schemas)。

###### 2. 配置maven的settings文件
&emsp;&emsp;到[华为镜像站](https://mirrors.huaweicloud.com/)里面找到HuaweiCloud SDK，下载settings.xml，替换掉原来的settings文件。在settings文件中增加如下配置。
``` xml
<profiles>
  <profile>
    <id>MyProfile</id>
    <pluginRepositories>
      <pluginRepository>
        <id>HuaweiCloudSDK</id>
        <url>https://repo.huaweicloud.com/repository/maven/huaweicloudsdk/</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
      </pluginRepository>
    </pluginRepositories>
  </profile>
</profiles>
```

###### 3. 配置pom文件，引入插件

使用eclipse的用户可能会发现pom文件出现“Plugin execution not covered by lifecycle configuration”的报错信息。出现这种情况不影响插件使用，可以不用理会，也可以按照eclipse的修复提示进行修复。

参数说明如下。

| 参数	| 说明 |
| - | - |
| skip | 是否跳过执行该插件功能，默认是true，所以这里需要手动将skip设为false。 |
| skipOverWrite	| 是否跳过文件覆盖，默认是false，即每次运行插件都可以更新框架代码。 |
| repositories|	定义多个契约仓库。 |
| repository |	定义单个契约仓库，即远端契约所在的git仓库。 |
| userName | 契约仓库的用户名（选填）。 |
| password	| 契约仓库密码（选填）。 |
| repoUrl	| 契约仓库地址，http、https、ssh格式都适用。 |
| branch |	契约仓库分支名。 |
| services |	定义多个服务。 |
| service	| 定义单个服务，每个服务可以有多个契约文件。 |
| appId	| 应用Id（选填，只在consumer这边指定，consumer跨应用调用provider的时候可以填对应的provider的appId）。 |
| serviceName	| 服务名(服务消费者consumer和服务提供者provider都填provider的服务名)。 |
| packageName（service层） | 生成的框架代码（delegate、impl、controller）的包路径，当契约中的model里面没有x-java-class，也作为model的包路径。 |
| schemaType | 指定服务是consumer还是provider，根据契约生成相应的框架代码。 |
| schemas |	定义多个契约文件。 |
| schema |	定义单个契约文件。 |
| schemaPath |	契约文件在契约仓库的相对路径。 |
| packageName	| 生成代码的包路径，优先级小于service里面的packageName，当两者都没有设置，插件运行会报错。 |

* consumer模块引入huawei-swagger-codegen-maven-plugin，插件版本号是2.2.8。schemaType指定参数“consumer”，生成服务消费者框架代码。
``` xml
<plugins>
    <plugin>
        <groupId>io.swagger</groupId>
        <artifactId>huawei-swagger-codegen-maven-plugin</artifactId>
        <version>2.2.8</version>
        <executions>
            <execution>
                <goals>
                    <goal>generate</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <skip>false</skip>
            <!--<skipOverwrite>false</skipOverwrite>-->
            <repositories>
                <repository>
                    <!--<userName></userName>-->
                    <!--<password></password>-->
                    <repoUrl> https://github.com/huaweicse/cse-codegen-schemas.git</repoUrl>
                    <branch>master</branch>
                    <services>
                        <service>
                            <!--<appId>lala</appId>-->
                            <serviceName>myprovider</serviceName>
                            <packageName>com.huawei.paas.consumer</packageName>
                            <schemaType>consumer</schemaType>
                            <schemas>
                                <schema>
                                    <schemaPath>dir/myservice.yaml</schemaPath>
                                </schema>
                            </schemas>
                        </service>
                    </services>
                </repository>
            </repositories>
            <packageName>com.huawei.paas.consumer</packageName>
        </configuration>
    </plugin>
</plugins>
```

* provider模块引入huawei-swagger-codegen-maven-plugin，插件版本号是2.2.8。schemaType指定参数“provider”，生成服务提供者框架代码。
``` xml
<plugins>
    <plugin>
        <groupId>io.swagger</groupId>
        <artifactId>huawei-swagger-codegen-maven-plugin</artifactId>
        <version>2.2.8</version>
        <executions>
            <execution>
                <goals>
                    <goal>generate</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <skip>false</skip>
            <skipOverwrite>false</skipOverwrite>
            <repositories>
                <repository>
                    <!--<userName></userName>-->
                    <!--<password></password>-->
                    <repoUrl> https://github.com/huaweicse/cse-codegen-schemas.git</repoUrl>
                    <branch>master</branch>
                    <services>
                        <service>
                            <serviceName>myprovider</serviceName>
                            <packageName>com.huawei.paas.provider</packageName>
                            <schemaType>provider</schemaType>
                            <schemas>
                                <schema>
                                    <schemaPath>dir/myservice.yaml</schemaPath>
                                </schema>
                            </schemas>
                        </service>
                    </services>
                </repository>
            </repositories>
            <packageName>com.huawei.paas.provider</packageName>
        </configuration>
    </plugin>
</plugins>
```

###### 4. 运行插件，同步契约并生成框架代码
执行命令，运行CSE-Codegen插件：mvn generate-sources  
或者直接编译整个微服务工程，运行插件。当然，不是说每次编译都要运行插件，所以插件配置提供了一个skip参数，默认skip为false，如果想要阻止插件运行导致重复生成框架代码，我们只需要将skip设置为true。

同步契约：
1. 首先会检查工程的gitRepo目录下是否存在同名的Git仓库，如果存在则进行删除。
2. 下载整个Git仓库到工程的gitRepo目录下。
3. 插件根据scchemaPath查找gitRepo里的契约文件，复制到契约文件到工程的microservices目录下，如目录中存在同名契约则替换掉。

生成框架代码：
1. 生成consumer端框架代码：model类、delegate接口、impl实现类（具备RPC调用接口）。
2. 生成provider端框架代码：model类、controller类、delegate接口、impl实现类。  

###### 5. 根据框架代码，用户实现自己的业务逻辑
如果impl目录不存在或者impl目录下不存在契约对应的impl实现类，则生成impl实现类，否则不生成（避免覆盖用户已有的业务逻辑）。用户可以在impl实现类中增加自己的业务逻辑。如果用户想要使用框架提供的impl实现类，那么就必须将现有的实现类删除，重新运行插件。

###### 6. 测试consumer和provider
测试consumer和provider是否可以通信，可以下载[本地服务中心](https://console.huaweicloud.com/cse/?region=cn-north-1#/cse/tools)，如下图所示。

![serviceCenterDownload](https://github.com/huaweicse/cse-java-chassis-samples/blob/master/cse-codegen-doc/serviceCenterDownload.png)

配置好工程里的microservice.yaml，启动服务中心，启动consumer和provider，测试consumer和provider是否正常通信。
