> Spring Cloud Bus配合Spring Cloud Config使用可以实现配置的动态刷新，目前只支持RabbitMQ和Kafka

##### 什么是总线

​	在微服务架构的系统中，通常会使用<font color="red">轻量级的消息代理</font>来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于<font color="red">该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线</font>。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

##### 基本原理

​	ConfigClient实例都监听MQ中同一个topic（默认是springCloudBus）。当一个服务刷新数据时，它会把这个消息放入topic中，这样其他监听同一topic的服务就能得到通知，然后去更新自身的配置。



## 一、RabbitMQ环境配置(windows)

#### 1. 安装Erlang

http://erlang.org/download/otp_win64_21.3.exe

#### 2. 安装RabbitMQ

https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe

#### 3. 启动管理功能

在RabbitMQ安装目录bin/下打开cmd:`rabbitmq-plugins enable rabbitmq_management`

#### 4. 启动并访

在开始菜单中启动 `RabbitMQ Service - start`

访问http://localhost:15672 用户名: guest 密码: guest



## 二、全局广播

### 1. 复制client3355，新增client3366

### 2. 配置中心center3344添加消息总线支持

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yaml

spring:
  #...
	rabbitmq:
  	host: localhost
  	port: 5672
  	username: guest
  	password: guest
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'  	
```



###  三、客户端center3355和client3366添加消息总线支持

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yaml
spring:
  #...
	rabbitmq:
  	host: localhost
  	port: 5672
  	username: guest
  	password: guest
```

### 四、测试

​	修改gitee上的配置文件后，通过`curl -X POST "http://localhost:3344/actuator/bus-refresh"`刷新`center3344`上的，订阅的客户端就会同步刷新。



## 三、定点通知

指定具体某一个实例生效，而不是全部[http://localhost:{port}/actuator/bus-refresh/{destination}]()。

如：http://localhost:3344/actuator/bus-refresh/config-client:3366

destination：{spring.application.name}:{server.port}