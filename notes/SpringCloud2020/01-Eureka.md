## 一、eureka基础知识

#### 1.1 什么是服务治理

​	传统的rpc调用框架中，管理每个服务之间的依赖关系比较复杂，所以需要<font color="blue">服务治理</font>，来管理这些依赖关系，可以实现服务调用负载均衡容错等，实现服务注册与发现

#### 1.2 什么是服务注册与发现

​	Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。

​	在服务注册与发现中，有一个注册中心。在服务器启动时，会把自己当前服务器的信息（如通讯地址等）以<font color="blue">别名</font>的方式注册到注册中心上。另一方（消费者|服务提供者），以该别名的方式去注册中心上<font color="blue">获取实际的服务通讯地址</font>，然后再进行远程调用。

#### 1.3 Eureka两个组件

- <font color="red">Eureka Server：</font>提供服务注册服务

  ​	各个微服务节点通过配置启动后，会在Eureka Server中进行注册，这样服务注册表中将会存储所有可用服务节点的信息，可以在界面上直观的看到。

- <font color="red">Eureka Client：</font>通过注册中心进行访问

  ​	是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的 使用轮询的(round-robin）负载均衡算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（<font color="blue">默认周期30秒</font>）。在多个周期内没有收到某个节点的心跳，会把这个节点从服务列表中移除（<font color="blue">默认90秒</font>）



## 二、单机Eureka构建

### 2.1 创建Eureka Server端服务注册中心

​	(1）创建注册中心module: <font color="red" size="5">`cloud-eureka-server7001`</font>
​	(2）pom.xml中引入eureka-server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
​	(3）配置application.yml文件

```yaml
server:
  port: 7001
eureka:
  instance:
    # eureka服务器的实例名称
    hostname: localhost
  client:
    # false表示不向服务器注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心：职责是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务 都需要依赖于这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```
​	(4）主启动类增加注解，标记为eureka服务端
```java
@SpringBootApplication(exclude= DataSourceAutoConfiguration.class）
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args） {
        SpringApplication.run(EurekaMain7001.class,args）;
    }
}
```

（5）启动后访问http://localhost:7001即可看到eureka服务启动成功。

![image-20200804213010538](https://gitee.com/liukai830/picgo/raw/master/image-20200804213010538.png)



### 2.2 提供者payment8001注册进注册中心

​	(1）pom.xml引入Eureka-client

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

​	(2）修改application.yml配置

```yaml
server:
  port: 8001
spring:
  application:
    name: cloud-payment-service #微服务的注册名称
eureka:
  client:
    # 是否将自己注册进Eureka Server,默认为true
    register-with-eureka: true
    # 是否从Eureka Server抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true，才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      # Eureka Server注册地址
      defaultZone: http://localhost:7001/eureka/
```
​	(3）主启动类标记为Eureka客户端

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {
    public static void main(String[] args） {
        SpringApplication.run(PaymentMain8001.class, args）;
    }
}
```



### 2.3 消费者consume-order80注册进注册中心

​	操作同上，微服务名为：

```yaml
spring:
  application:
    name: cloud-order-service
```



### 2.4 查看微服务注册情况

​	登录http://locathost:7001查看：可以看到提供者`CLOUD-PAYMENT-SERVICE`和消费者`CLOUD-ORDER-SERVICE`两个微服务已经注册进注册中心

![image-20200308181442071](https://gitee.com/liukai830/picgo/raw/master/image-20200308181442071.png)



##  三、集群Eureka构建

### 3.1 Eureka集群原理说明

​	互相注册，相互守望：集群中的任意一个client节点，会注册其他所有client节点

### 3.2 Eureka Server集群构建

​	(1）修改host文件（C:\Windows\System32\drivers\etc），增加域名映射

	127.0.0.1		eureka7001.com
	127.0.0.1		eureka7002.com

​	(2）创建注册中心module: <font color="red" size="5">`cloud-eureka-server7002`</font>

​	(3）修改eureka7001 yml配置，<font color="blue">修改实例名和注册的地址: </font>7001注册7002，7002注册7001

```yaml
server:
  port: 7001
eureka:
  instance:
    # eureka服务器的实例名称
    hostname: eureka7001.com
  client:
    # false表示不向服务器注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心：职责是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      # 设置与Eureka Server交互的地址查询服务和注册服务 都需要依赖于这个地址
      defaultZone: http://eureka7002.com:7002/eureka/
```

​	（4）验证：

​	登录 http://eureka7001.com:7001/ 和 http://eureka7002.com:7002/ 能在注册列表中看到其他的server节点即为配置集群成功。

![image-20200308192710509](https://gitee.com/liukai830/picgo/raw/master/image-20200308192710509.png)

### 3.3 将支付服务8001和订单服务80发布到Eureka集群中

​	（1）修改两个工程的application.yml配置: (只显示被修改的部分)

```yaml
eureka:
  client:
    service-url:
      # 单机版本
      # defaultZone: http://localhost:7001/eureka/ 
      # 集群版本，有几个server就写几个地址
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka 
```

​	（2）验证

​	登录 http://eureka7001.com:7001/ 和 http://eureka7002.com:7002/ 都可以看到payment和order两个微服务即为成功。

![image-20200804215040611](https://gitee.com/liukai830/picgo/raw/master/image-20200804215040611.png)

![image-20200804215123624](https://gitee.com/liukai830/picgo/raw/master/image-20200804215123624.png)

### 3.4 支付服务提供者8001集群环境构建

​	增加一个同样的module：<font color="red" size="5">`cloud-prodiver-payment8002`</font>，

​	可以看到在微服务**CLOUD-PAYMENT-SERVICE**下有**8001**和**8002**两个节点，只对外暴露一个微服务名称。

![image-20200804215712271](https://gitee.com/liukai830/picgo/raw/master/image-20200804215712271.png)

![image-20200804215731217](https://gitee.com/liukai830/picgo/raw/master/image-20200804215731217.png)

### 3.5 负载均衡

#### 3.5.1 发现问题

​	**修改consume80 restTemplate远程调用为服务提供者的url地址后，在postman中重新访问80端口下的请求。**

```java
// public static final String PAYMENT_URL = "http://localhost:8001";
public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
```

​	**发现有报错：**<font color="red">同一个微服务下有多个节点，程序不能通过微服务名知道到底是哪一个微服务进行服务提供</font>

![image-20200308203122056](https://gitee.com/liukai830/picgo/raw/master/image-20200308203122056.png)
​	

#### 3.5.2 解决问题

​	开启restTemplate的负载均衡功能：

​	修改``cloud-consume-order80``微服务config，<font color="red">增加`@LoadBalanced`注解，赋予RestTemplate负载均衡的能力</font>

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced   // 使用该注解赋予RestTemplate负载均衡的能力(Ribbon提供的客户端)
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

}
```



##  四、actuator微服务完善

​	**（1）微服务注册列表中只显示服务名，不显示主机 端口信息：增加application.yml配置**

```yaml
eureka:
  instance:
    instance-id: payment8001
```
![image-20200308205531955](https://gitee.com/liukai830/picgo/raw/master/image-20200308205531955.png)

​	**（2）鼠标悬停在微服务上显示IP信息**

```yaml
eureka:
  instance:
    prefer-ip-address: true
```
![image-20200308210146920](https://gitee.com/liukai830/picgo/raw/master/image-20200308210146920.png)



## 五、服务发现Discovery

​	对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息。

（1）在主启动类中增加`@EnableDiscoveryClient`注解

（2）在controller写方法查询已注册的微服务相应信息

```java
import org.springframework.cloud.client.discovery.DiscoveryClient;

@Autowired private DiscoveryClient discoveryClient;

@PostMapping("/discovery")
public R discovery() {
    // 查询所有微服务
    log.info("-----  all services  -----");
    List<String> services = discoveryClient.getServices();
    services.forEach(service -> log.info(service));

    log.info("-----  instances of CLOUD-PAYMENT-SERVICE  -----");
    // 查询一个微服务下所有实例
    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    instances.forEach(
      serviceInstance ->
        log.info(
          "serviceId:" + serviceInstance.getServiceId()
          + "host: " + serviceInstance.getHost()
          + "uri: " + serviceInstance.getUri()
      )
    );
    return new R(200,"查询微服务信息成功！");
}
```
​	查看日志输出：

![image-20200308212700475](https://gitee.com/liukai830/picgo/raw/master/image-20200308212700475.png)



##  六、eureka自我保护

#### 6.1 自我保护理论

Eureka Server将会尝试删除其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。 =>  <font color="blue">某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存。</font>

​	属于[CAP理论](https://www.jianshu.com/p/a1a67dd70194)中的<font color="#42b983">**AP**</font>：[一致性](https://baike.baidu.com/item/一致性/9840083)（Consistency） [可用性](https://baike.baidu.com/item/可用性/109628)（Availability） 分区容错性（Partition tolerance）

#### 6.2 关闭自我保护

​	服务端：在`eureka7001`和`eureka7002`的`application.yml`增加 eureka 配置

```yaml
eureka:
  server:
    # 关闭自我保护机制
    enable-self-preservation: false
    # 清理无效节点时间
    eviction-interval-timer-in-ms: 2000
```

​	客户端：在`payment8001`等客户端 `application.yml`增加 eureka 配置

```yaml
eureka:
  instance:
    # Eureka客户端向服务端发送心跳的时间间隔，单位为秒（默认30秒）
    lease-renewal-interval-in-seconds: 1
    # Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒（默认是90秒），超时将剔除服务（如果关闭了自我保护）
    lease-expiration-duration-in-seconds: 2
```