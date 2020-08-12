## 一、概念简述

​	Spring Cloud Gateway是Spring Cloud的一个全新项目，该项目是基于spring5.0，spring boot 2.0 和 project reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的API路由管理方式。
​	Spring Cloud Gateway 作为spring 丑陋的生态系统中的网关，目标是替换zetflix zuul ,其中不经提供统一的路由方式，并且久雨filter 链的方式提供了网关基本的功能,例如：安全，监控，埋点和限流等。
​	<font color="blue">Spring Cloud Gateway使用的Webflux中的creator-netty响应式编程组件，底层使用了Netty通讯框架。</font>

![image-20200809195352005](https://gitee.com/liukai830/picgo/raw/master/image-20200809195352005.png)



### 1.1 Gateway

​	Gateway是基于**<font color="red">异步非阻塞模型</font>**上进行开发的，性能方面强。具有如下特性：

- 基于Spring Freamwork5.0，Spring Boot 2.0 和 Broject Reactor 进行构建；
- 动态路由：能够匹配任何请求属性；
- 可以对路由指定Predicate（断言）和Filter（过滤器）；
- 集成Hystrix的断路器功能；
- 集成Spring Cloud服务发现功能；
- 易于编写的Predicate（断言）和Filter（过滤器）；
- 请求限流功能；
- 支持路径重写



### 1.2 Gatewal & Zuul 1.x

​	在SpringCloud Finchley正式版之前，SpringCloud推荐的网关是Netflix提供的Zuul：

1. Zuul 1.x，是一个基于阻塞 I/O 的 API Gateway。
2. Zuul 1.x 基于Servlet 2.5使用阻塞架构，它不支持任何长连接（如WebSocket）。Zuul的设计模式和Nginx比较像，每次 I/O 操作都是从工作线程中选择一个执行，请求线程阻塞到工作线程完成，但是差别时Nginx用C++实现，Zuul用Java实现，而JVM本身会有第一次加载较慢的情况，使得Zuul的性能相对较差。
3. Zuul 2.x理念更先进，想基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。Zuul 2.x的性能较Zuul 1.x有较大提升。在性能方面，根据官方提供的基准测试，Spring Cloud Gateway的RPS（每秒请求数）是Zuul的1.6倍。
4. Spring Cloud Gateway 建立在Spring Freamwork5.0，Spring Boot 2.0 和 Broject Reactor 之上，使用非阻塞API。
5. Spring Cloud Gateway 还支持 WebSocket，并与Spring紧密集成拥有更好的开发体验。



## 二、三大核心概念

### 2.1 路由 Route
​	路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。
### 2.2 断言 Predicate
​	参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由。
### 2.3 过滤 Filter
​	指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。
### 2.4 总体
![20200318202547900](https://gitee.com/liukai830/picgo/raw/master/20200318202547900.png)
​	Web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化的控制。
​	Predicate就是我们的匹配条件；而Filter就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了。



## 三、Gateway工作流程

<center class="half">
<img src="https://gitee.com/liukai830/picgo/raw/master/无标题.png" width="450"/>
<img src="https://gitee.com/liukai830/picgo/raw/master/12191355-28ebd217899aa37e.png" width="550"/>
</class>



​	客户端向Spring Cloud Gateway 发出请求，然后在Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway Web Handler。
​	Handler再通过指定的过滤连将请求大宋我实际的服务执行业务，然后返回。过滤器质检用虚线分开是因为过滤器可能会在发送代理请求之前("pre")或之后("post")执行业务逻辑。
​	Filter在"pre"类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等；在"post"类型的过滤器中可以做响应内容、响应头的修改、日志的输出、流量监控等有着非常重要的作用。





## 四、入门配置

### 1. 创建cloud-gateway-gateway9527

#### pom

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>com.liuk.springcloud</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    </dependencies>
```



#### application.yml

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由
        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

#### 测试

加入网关前：http://localhost:8001/payment/get/1

加入网关后：http://localhost:9527/payment/get/1

<center class="half">
<img src="https://gitee.com/liukai830/picgo/raw/master/image-20200812201803597.png"/>
<img src="https://gitee.com/liukai830/picgo/raw/master/image-20200812201844552.png"/>
</class>



### 2. 路由配置的两种方式

#### （1）application.yml

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由
        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由
```



#### （2）硬编码

```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        return  routeLocatorBuilder.routes()
                .route("my_route_a", r -> r.path("/guonei").uri("http://news.baidu.com/guonei"))
                .route("my_route_a", r -> r.path("/guoji").uri("http://news.baidu.com/guoji")).build();
    }
}
```





## 五、通过微服务名实现动态路由

​	默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由功能，利用微服务名进行路由
      routes:
        - id: payment_routh
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/**
        - id: payment_routh
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**
```



## 六、Predicate的使用

![12191355-7c74ff861a209cd9](https://gitee.com/liukai830/picgo/raw/master/12191355-7c74ff861a209cd9.png)



>  Route predicate Factories 路由谓词工厂

#### 1. After Route Predicate Factory

​	此谓词匹配当前日期时间之后发生的请求。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: lb://cloud-payment-service
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

#### 2. Before Route Predicate Factory

​	此谓词匹配在当前日期时间之前发生的请求。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: lb://cloud-payment-service
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

#### 3. Between Route Predicate Factory

​	此谓词匹配datetime1之后和datetime2之前发生的请求。 datetime2参数必须在datetime1之后。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: lb://cloud-payment-service
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

#### 4. Cookie Route Predicate Factory

​	Cookie Route Predicate Factory有两个参数，cookie名称和正则表达式。此谓词匹配具有给定名称且值与正则表达式匹配的cookie。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: lb://cloud-payment-service
        predicates:
        - Cookie=chocolate, ch.p
```

#### 5. Header Route Predicate Factory

​	Header Route Predicate Factory有两个参数，标题名称和正则表达式。此谓词与具有给定名称且值与正则表达式匹配的标头匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: lb://cloud-payment-service
        predicates:
        - Header=X-Request-Id, \d+
```

#### 6. Host Route Predicate Factory

​	Host Route Predicate Factory采用一个参数：主机名模式。该模式是一种Ant样式模式“.”作为分隔符。此谓词匹配与模式匹配的Host标头。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: lb://cloud-payment-service
        predicates:
        - Host=**.somehost.org
```

#### 7. Method Route Predicate Factory

​	Method Route Predicate Factory采用一个参数：要匹配的HTTP方法。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: lb://cloud-payment-service
        predicates:
        - Method=GET
```

#### 8. Path Route Predicate Factory

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: lb://cloud-payment-service
        predicates:
        - Path=/foo/{segment}
```

#### 9. Query Route Predicate Factory

​	Query Route Predicate Factory有两个参数：一个必需的参数和一个可选的正则表达式。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: lb://cloud-payment-service
        predicates:
        - Query=baz		# 如果请求包含baz查询参数，则此路由将匹配。
```

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: lb://cloud-payment-service
        predicates:
        - Query=foo, ba.	# 如果请求包含其值与ba匹配的foo查询参数，则此路由将匹配。 regexp，所以bar和baz匹配。
```

#### 10. RemoteAddr Route Predicate Factory

​	RemoteAddr Route Predicate Factory采用CIDR符号（IPv4或IPv6）字符串的列表（最小值为1），例如， 192.168.0.1/16（其中192.168.0.1是IP地址，16是子网掩码）。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: lb://cloud-payment-service
        predicates:
        - RemoteAddr=192.168.1.1/24
```



## 七、Filter的使用

> 路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用
> Spring Cloud Gateway 内置了多种路由过滤器，他们都有GatewayFilter的工厂类来产生

- Filter生命周期
  - pre：在业务逻辑之前
  - post：在业务逻辑之后
- 种类
  - GatewayFilter：单一
  - GlobalFilter：全局



### 自定义全局GlobalFilter

​	只要实现`GlobalFilter`和`Ordered`接口即可；自定义过滤器可以用于`全局日志记录`、`统一网关鉴权`、……

```java
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("*********come in MyLogGateWayFilter: " + LocalDateTime.now());
        String uname = exchange.getRequest().getQueryParams().getFirst("username");
        if(StringUtils.isBlank(uname)) {
            log.info("*****用户名为Null 非法用户");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
  
    /* 过滤器优先级，数字越小优先级越高 */
    @Override
    public int getOrder() {
        return 0;
    }
}
```











