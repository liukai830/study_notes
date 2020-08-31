> Spring Cloud Conig是为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为<font color="red">各个不同微服务应用</font>的所有环境提供了一个<font color="red">中心化的外部配置</font>

​	Spring Cloud Config分为<font color="red">服务端</font>和<font color="red">客户端</font>两部分。

- 服务端

  也称<font color="red">分布式配置中心，它是一个独立的微服务应用</font>，用来连接配置服务器并为客户端提供获取配置信息，加密解密信息等访问接口。

- 客户端

  是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载信息配置服务器。默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。



## 一、服务端配置

### 1. 创建配置仓库

​	在gitee上创建[springcloudconfig](https://gitee.com/liukai830/springcloudconfig)配置仓库，添加conf-xxx.yml配置文件，内容如下：

```yaml
config:
  info: "mater branch,springcloud-config/config-dev.yml version=1"
```

![image-20200813205514231](https://gitee.com/liukai830/picgo/raw/master/image-20200813205514231.png)



### 2. 创建cloud-config-center3344

#### （1）pom.xml

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### （2）application.yml

```yaml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri:  https://gitee.com/liukai830/springcloudconfig	#仓库地址
          search-paths:
            - springcloudconfig	#读取的目录
      label: master	# 分支
eureka:
  client:
    service-url:
      defaultZone:  http://localhost:7001/eureka
```

#### （3）主启动类

加注解`@EnableConfigServer`



### 3. 测试

​	http://localhost:3344/config-dev.yml 查看是否能读取到配置中心的配置信息。

![image-20200813205712196](https://gitee.com/liukai830/picgo/raw/master/image-20200813205712196.png)

### 4.配置读取规则

/{name}-{profiles}.yml
/{label}-{name}-{profiles}.yml

- label：分支(branch)
- name：服务名
- profiles：环境(dev/test/prod)



![image-20200813210719162](https://gitee.com/liukai830/picgo/raw/master/image-20200813210719162.png)



## 二、客户端配置

### 1. 创建cloud-config-client-3355

####（1）pom.xml

```xml
<!-- 客户端和服务端config的jar包不一样！！！ -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### （2）bootstrap.yml

- application.yml：用户级的资源配置项
- bootstrap.yml：系统级，优先级更高

​    Spring Cloud会创建一个"Bootstrap Context"，作为Spring应用的"Application Context"的父上下文。初始化的时候，"Bootstrap Context"负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的"Environment"。

​    "Bootstrap"属性有高优先级，默认情况下，它们不会被本地配置覆盖。"Bootstrap Context"和"Application Context"有着不同的约定，所以新增了"bootstrap.yml"，保证"Bootstrap Context"和"Application Context"配置的分离。

​	要将Client模块下的application.yml文件改为bootstrap.yml，这是很关键的。因为bootstrap.yml是比application.yml先加载的。bootstrap.yml优先级高于application.yml。



bootstrap.yml内容如下：

```yaml
server:
  port: 3355
spring:
  application:
    name: config-client
  cloud:
    config: # config客户端配置
      label: master	# 分支
      name: config	# 配置文件名称
      profile: dev	# 读取后缀名称
      uri: http://localhost:3344	# 配置中心地址
      # 上述综合，master分支上config-dev.yml的配置文件会被读取
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```



#### （3）主启动类及业务类

```java
/******** 主启动类 ********/
@SpringBootApplication
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}

/******** 业务类 ********/
@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;
    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```



#### （4）测试

http://localhost:3355/configInfo 通过RESTful接口能查询到3355的配置信息。

![image-20200813213848646](https://gitee.com/liukai830/picgo/raw/master/image-20200813213848646.png)



## 三、动态刷新

1. 在启动服务端和客户端后，测试都能正常读取到配置信息。
2. gitee上修改dev配置 version=2
3. 查看服务端和客户端获取的配置信息
4. 发现服务端获取到修改后的配置信息，客户端还是原来的配置信息

![image-20200813214717666](https://gitee.com/liukai830/picgo/raw/master/image-20200813214717666.png)

#### （1）客户端增加pom

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### （2）修改客户端bootstrap.yml

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

####（3）@RefreshScope

```java
@RestController
@RefreshScope	// 增加的注解，用于刷新配置信息
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

#### （4）手动刷新

1. 修改配置后，查看服务端配置已刷新，客户端配置未刷新

2. 发送`POST`请求：http://localhost:3355/actuator/refresh

   ```bash
   C:\Users\liuk>curl -X POST http://localhost:3355/actuator/refresh
   ["config.client.version","config.info"]
   ```

3. 再次查看客户端就能看到配置刷新了



