##  What is OpenFeign

​	Feign是一个声明式的web服务客户端，让编写web服务客户端变得非常容易，只需<font color="red">创建一个接口并在接口上添加注解即可</font>。

#### Feign能干嘛

​	Feign旨在使编写Java Http客户端变得更容易。

​	前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，<font color="red">往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用</font>。所以，Feign在此基础上作了进一步的封装，由它来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，<font color="red">我们只需要创建一个接口并使用注解的方式来配置它（以前是在Dao接口上标注Mapper注解，现在是一个微服务上标注一个Feign注解即可）</font>，即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon时，自动封装服务调用客户端的开发量。

#### Feign集成了Ribbon

​	利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，<font color="red">通过feign只需要定义服务绑定接口且以声明式的方法</font>，优雅而简单的实现了服务调用。

#### Feign & OpenFeign

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 是SpringCloud组件中的一个轻量级的RESTful的HTTP服务客户端。内置了Ribbon，用来做客户端负载均衡，去调用注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务。 | 是SpringCloud在Feignd的基础上支持了SpringMVC的注解，如@RequestMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |
| <dependency>     <groupId>org.springframework.cloud</groupId>     <artifactId>spring-cloud-starter-openfeign</artifactId> </dependency> | <dependency>     <groupId>org.springframework.cloud</groupId>     <artifactId>spring-cloud-starter-openfeign</artifactId> </dependency> |



## 一、OpenFeign使用

**<font color="red">Feign在消费端使用</font>**

### 1. 新建`cloud-consumer-order80-feign`

#### pom.xml增加openfeign

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### application.yml

```yaml
server:
  port: 80
spring:
  application:
    name: cloud-order-service # 微服务的名称
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

#### OrderFeignMain80

```java
package com.liuk.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class, args);
    }
}
```

#### service

```java
package com.liuk.cloud.service;
import com.liuk.cloud.entity.CommonResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
    @GetMapping(value = "payment/get/{id}")
    CommonResult getPaymentById(@PathVariable("id") Long id);
}
```

####controller

```java
package com.liuk.cloud.controller;

import com.liuk.cloud.entity.CommonResult;
import com.liuk.cloud.entity.Payment;
import com.liuk.cloud.service.PaymentFeignService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
@RestController
@Slf4j
public class OrderFeignController {
    @Autowired
    private PaymentFeignService paymentFeignService;
    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
        return paymentFeignService.getPaymentById(id);
    }
}
```

####验证

访问http://localhost/consumer/payment/get/1即可验证，而且可以看出feign自带负载均衡功能。

#### 注意事项

主启动类`@EnableFeignClients`和feign接口`@FeignClient(value = "CLOUD-PAYMENT-SERVICE")`

![image-20200808162225672](https://gitee.com/liukai830/picgo/raw/master/image-20200808162225672.png)



## 二、OpenFeign超时控制

### 1. 超时错误复现

#### （1）在8001和8002controller中增加超时的方法

```java
 @GetMapping(value = "payment/timeout")
 public String timeout(){
   try {
     TimeUnit.SECONDS.sleep(3);
   } catch (InterruptedException e) {
     e.printStackTrace();
   }
   return serverPort;
 }
```

#### （2）cloud-consumer-order80-feign调用

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
    @GetMapping(value = "payment/timeout")
    CommonResult timeout();
}
```

```java
@RestController
@Slf4j
public class OrderFeignController {
    @Autowired
    private PaymentFeignService paymentFeignService;
    @GetMapping(value = "/consumer/payment/timeout")
    public CommonResult<Payment> timeout() {
        return paymentFeignService.timeout();
    }
}
```

#### （3）结果

​	直接请求http://localhost:8001/payment/timeout，经过3s后返回端口号8001；

​	直接请求http://localhost/consumer/payment/timeout，经过1s后返回错误信息。

​	这是因为**<font color="red">OpenFeign默认等待一秒钟，超过后报错</font>**

![image-20200808164155808](https://gitee.com/liukai830/picgo/raw/master/image-20200808164155808.png)



### 2、开启OpenFeign客户端超时控制

在application.yml配置中增加

```yaml
server:
  port: 80
spring:
  application:
    name: cloud-order-service # 微服务的名称
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
ribbon:
	# 建立连接后从服务器读取到可用资源所用的时间
  ReadTimeout: 5000
  # 建立连接所用的时间，适用于网络状况正常的情况下，两端连接所用时间
  ConnectTimeout: 5000
```





## 三、OpenFeign日志打印

​	对Feign接口的调用情况进行监控和输出。

### 1. 配置FeignConfig

日志级别：

- NONE：默认的，不显示任何日志
- BASIC：仅记录请求方法、URL、响应状态码以及执行世家
- HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息
- FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文以及元数据



```java
package com.liuk.cloud.config;
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

### 2. 修改yml

YML文件里需要开启日志的Feign客户端

```yaml
server:
  port: 80
spring:
  application:
    name: cloud-order-service # 微服务的名称
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ReadTimeout: 5000
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ConnectTimeout: 5000
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.liuk.cloud.service.PaymentFeignService: debug
```

### 3. 后台日志查看

请求http://localhost/consumer/payment/get/1，可以查看到后台的详细日志。

![image-20200808170206304](https://gitee.com/liukai830/picgo/raw/master/image-20200808170206304.png)