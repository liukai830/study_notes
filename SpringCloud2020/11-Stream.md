## 一、消息驱动概述

是什么：屏蔽底层消息中间件的差异，降低切换版本，统一消息的编程模型。

应用程序通过@Input或者@Output来与Spring Cloud Stream中的binder对象交互，目前仅支持RabbitMQ、Kafka。

### 1.1 设计思想

#### 标准MQ：

- 生产者/消费者之间靠消息媒介传递消息内容：Message

- 消息必须走特定的通道：MessageChannel

- 消息通道MessageChannel的子接口SubScribableChannel，由MessageHandler消息处理器订阅

#### 为什么用Cloud Stream：
​	在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们在实现细节上会有较大的差异性。
​	通过定义绑定器作为中间层，完美的实现了<font color="red">应用程序与消息中间件细节之间的隔离</font>。通过向应用程序暴露统一的Channel通道，应用程序不需要再考虑各种不同的消息中间件实现。
​	<font color="red">通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离</font>

- Input：注解标识输入通道，接收（消息消费者）的消息将通过该通道进入应用程序
- Output：注解标识输出通道，发布（消息生产者）的消息将通过该通道离开应用程序。

#### 发布-订阅：

Stream中的消息通信方式遵循了发布-订阅模式：

Topic主题进行广播：

- 在RabbitMQ就是Exchange
- 在Kafka就是Topic



### 2.2 工作流程

- Binder
  方便的连接中间件，屏蔽差异
- Channel
  通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过对Channel对队列进行配置
- Source和Sink
  简单的可以理解为参照对象就是Spring Cloud Stream本身，从Stream发布消息就是输出，接收消息就是输入

![171c8b790feb0e77](https://gitee.com/liukai830/picgo/raw/master/171c8b790feb0e77.png)



　　该模型图中有如下几个核心概念：

- `Source`：当需要发送消息时，我们就需要通过 `Source.java`，它会把我们所要发送的消息进行序列化（默认转换成 JSON 格式字符串），然后将这些数据发送到 Channel 中；
- `Sink`：当我们需要监听消息时就需要通过 `Sink.java`，它负责从消息通道中获取消息，并将消息反序列化成消息对象，然后交给具体的消息监听处理；
- `Channel`：通常我们向消息中间件发送消息或者监听消息时需要指定主题（Topic）和消息队列名称，一旦我们需要变更主题的时候就需要修改消息发送或消息监听的代码。通过 `Channel` 对象，我们的业务代码只需要对应 `Channel` 就可以了，具体这个 Channel 对应的是哪个主题，可以在配置文件中来指定，这样当主题变更的时候我们就不用对代码做任何修改，从而实现了与具体消息中间件的解耦；
- `Binder`：通过不同的 `Binder` 可以实现与不同的消息中间件整合，`Binder` 提供统一的消息收发接口，从而使得我们可以根据实际需要部署不同的消息中间件，或者根据实际生产中所部署的消息中间件来调整我们的配置。



### 2.3 常用注解

<center class="half">
<img src="https://gitee.com/liukai830/picgo/raw/master/l.png">
<img src="https://gitee.com/liukai830/picgo/raw/master/r.png">
</center>
| 组成            | 说明                                                         |
| :-------------- | :----------------------------------------------------------- |
| Middleware      | 中间件，支持 RabbitMQ 和 Kafka。                             |
| Binder          | 目标绑定器，目标指的是 Kafka 还是 RabbitMQ。绑定器就是封装了目标中间件的包。如果操作的是 Kafka 就使用 `spring-cloud-stream-binder-kafka`，如果操作的是 RabbitMQ 就使用 `spring-cloud-stream-binder-rabbit`。 |
| @Input          | 注解标识输入通道，接收（消息消费者）的消息将通过该通道进入应用程序。 |
| @Output         | 注解标识输出通道，发布（消息生产者）的消息将通过该通道离开应用程序。 |
| @StreamListener | 监听队列，消费者的队列的消息接收。                           |
| @EnableBinding  | 注解标识绑定，将信道 `channel` 和交换机 `exchange` 绑定在一起。 |



## 二、案例说明

- 准备RabbitMQ环境
- 新建三个子模块
  - cloud-stream-rabbitmq-provider8801：生产者进行发送消息模块
  - cloud-stream-rabbitmq-provider8802：消息接收模块
  - cloud-stream-rabbitmq-provider8803：消息接收模块



## 三、消息驱动之生产者

创建子模块`cloud-stream-rabbitmq-provider8801`

#### 3.1 pom.xml

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```



#### 3.2 application.yml

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```



#### 3.3 主启动类StreamMQMain8001

```java
@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8801.class, args);
    }
}
```



#### 3.4 发送消息接口及实现类

```java
/* 接口 */
public interface MessageProvider {
    String send();
}

/* 实现类 */

@EnableBinding(Source.class)
public class MessageProdiverImpl implements MessageProvider {
    @Resource
    private MessageChannel output;
    @Override
    public String send() {
        String message = UUID.randomUUID().toString();
        boolean sendFlag = output.send(MessageBuilder.withPayload(message).build());
        System.out.println("message:" + message);
        return message;
    }
}
```



#### 3.5 controller

```java
@RestController()
@RequestMapping("/stream")
public class SendMessageController {
    @Autowired private MessageProvider messageProvider;
    @GetMapping("/send")
    public String send() { return  messageProvider.send();}
}
```



#### 3.6 测试

启动eureka7001、rabbitmq、stream8001，在rabbitmq WEB界面可以看到新的交换机`studyExchange`，来源于配置文件；

请求http://localhost:8801/stream/send，看到控制台打印出请求日志，WEB界面有波峰流量变动消息发送记录。

<center class="half">
<img src="https://gitee.com/liukai830/picgo/raw/master/image-20200913153634186.png" width="550">
<img src="https://gitee.com/liukai830/picgo/raw/master/image-20200913153813683.png"  width="500">
</center>





## 四、消息驱动之消费者

创建子模块cloud-stream-rabbitmq-provider8802

#### 4.1 pom.xml

同 3.1

#### 4.2 application.yml

与 3.2 相比，修改了spring.cloud.stream.bindings.<font color="red">output</font> => spring.cloud.stream.bindings.<font color="red">input</font>

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

#### 4.3 主启动类

同 3.3

#### 4.4 业务类

```java
@Component
@EnableBinding(Sink.class)
public class ReceiveMessageLinstnerController {
    @Value("${server.port}")
    private String port;
    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("接收到消息 -----> " + port + ": " + message.getPayload());
    }
}
```



#### 4.5 测试

通过 3.6 测试接口发送消息，在8802控制台查看接收的消息。 

![image-20200913161500491](https://gitee.com/liukai830/picgo/raw/master/image-20200913161500491.png)

![image-20200913161434837](https://gitee.com/liukai830/picgo/raw/master/image-20200913161434837.png)





## 五、分组消费与持久化

先仿8802创建8803模块，此时会有两个问题：
- 重复消费问题
- 消息持久化问题



#### 5.1 分组消费

Stream中处于同一个group中的多个消费者是竞争关系，就能够保证消息只会被其中一个应用消费一次。

- 不同组是可以全面消费的（重复消费）
- 同一组内会发生竞争关系，只有其中一个可以消费



修改配置文件增加group: groupA || groupB

```yaml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: groupA # 8002为groupA 8803为groupB
```



可以看到自定义的组名，如果需要避免重复消费，只需把group改为一致即可。

![image-20200913164827569](https://gitee.com/liukai830/picgo/raw/master/image-20200913164827569.png)



设置同一分组后：

![image-20200913165132601](https://gitee.com/liukai830/picgo/raw/master/image-20200913165132601.png)

![image-20200913165211849](https://gitee.com/liukai830/picgo/raw/master/image-20200913165211849.png)

![image-20200913165313843](https://gitee.com/liukai830/picgo/raw/master/image-20200913165313843.png)



#### 5.2 消息持久化

（1）先停止8802、8803；去掉8802的group:groupA

（2）8801发送4条消息到rabbitmq

（3）先启动8802，发现控制台没有打印消息

（4）再启动8803，发现控制台打印消息

结论：配置了group 属性后自动会持久化





## 六、自定义消息通道

#### 6.1 创建消息通道

参考源码 `Source.java` 和 `Sink.java` 创建自定义消息通道。

自定义消息发送通道 `MySource.java`

```java
package com.liuk.cloud.produce;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;
/**
 * 自定义消息发送通道
 */
public interface MySource {
    String MY_OUTPUT = "my_output";

    @Output(MY_OUTPUT)
    MessageChannel myOutput();
}
```

自定义消息接收通道 `MySink.java`

```java
package com.liuk.cloud.consumer;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;
/**
 * 自定义消息接收通道
 */
public interface MySink {
    String MY_INPUT = "my_input";

    @Input(MY_INPUT)
    SubscribableChannel myInput();
}
```



#### 6.2 配置文件

　消息生产者

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
        my_output: # 这个名字是一个通道的名称
          destination: liukMessage # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

消息消费者。

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          group: groupA
        my_input: # 这个名字是一个通道的名称
          destination: liukMessage # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```



#### 6.3 代码重构

消息生产者 `MyMessageProducer.java`。

```java
package com.liuk.cloud.service.impl;

import com.liuk.cloud.produce.MySource;
import com.liuk.cloud.service.MyMessageProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.messaging.support.MessageBuilder;

import java.util.UUID;

@EnableBinding(MySource.class)
public class MyMessageProviderImpl implements MyMessageProvider {

    @Autowired
    private MySource mySource;

    @Override
    public String send() {
        String message = "MY-" + UUID.randomUUID().toString();
        boolean sendFlag = mySource.myOutput().send(MessageBuilder.withPayload(message).build());
        System.out.println("message:" + message);
        return message;
    }
}
```

消息消费者 `MyMessageConsumer.java`。

```java
package com.liuk.cloud.controller;

import com.liuk.cloud.consumer.MySink;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(MySink.class)
public class MyReceiveMessageLinstnerController {
    @Value("${server.port}")
    private String port;

    @StreamListener(MySink.MY_INPUT)
    public void input(Message<String> message) {
        System.out.println("接收到消息 -----> " + port + ": " + message.getPayload());
    }
}
```



#### 6.4 测试

![image-20200913203047945](https://gitee.com/liukai830/picgo/raw/master/image-20200913203047945.png)

![image-20200913203115439](https://gitee.com/liukai830/picgo/raw/master/image-20200913203115439.png)

##### 访问

启动消息消费者，运行单元测试，消息消费者控制台打印结果如下：

```
接收到消息 -----> 8802: MY-4ec548ed-0e2a-4f62-8190-30d03e6a1623
接收到消息 -----> 8802: MY-7dd5a7f9-960f-416e-a801-4bad76d5828d
接收到消息 -----> 8802: MY-01090803-0342-4246-acac-0d65dc423952
接收到消息 -----> 8802: MY-59b97e07-9b6b-4b0a-82a7-ce651fd412e7
接收到消息 -----> 8802: MY-692b3506-a679-4193-9900-a32a7f812655
```

RabbitMQ 界面如下：

![image-20200913203248767](https://gitee.com/liukai830/picgo/raw/master/image-20200913203248767.png)





## 七、配置优化

​	Spring Cloud 微服务开发之所以简单，除了官方做了许多彻底的封装之外还有一个优点就是`约定大于配置`。开发人员仅需规定应用中不符约定的部分，在没有规定配置的地方采用默认配置，以力求最简配置为核心思想。

​	简单理解就是：Spring 遵循了推荐默认配置的思想，当存在特殊需求时候，自定义配置即可否则无需配置。

　在 Spring Cloud Stream 中，`@Output("output")` 和 `@Input("input")` 注解的 `value` 默认即为绑定的交换机名称。所以自定义消息通道的案例我们就可以重构为以下方式。

#### 7.1 创建消息通道

参考源码 `Source.java` 和 `Sink.java` 创建自定义消息通道。

　　自定义消息发送通道 `MySource02.java`

```java
package com.liuk.cloud.produce;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

/**
 * 自定义消息发送通道
 */
public interface MySource02 {

    String MY_OUTPUT = "default.message";
    @Output(MY_OUTPUT)
    MessageChannel myOutput();
}
```

　　自定义消息接收通道 `MySink02.java`

```java
package com.liuk.cloud.consumer;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

/**
 * 自定义消息接收通道
 */
public interface MySink02 {
    String MY_INPUT = "default.message";
    @Input(MY_INPUT)
    SubscribableChannel myInput();
}
```



#### 7.2 配置文件

​		消息生产者。

```yaml
server:
  port: 8801 # 端口
spring:
  application:
    name: stream-producer # 应用名称
  rabbitmq:
    host: 192.168.10.101  # 服务器 IP
    port: 5672            # 服务器端口
    username: guest       # 用户名
    password: guest       # 密码
    virtual-host: /       # 虚拟主机地址
```

　　　消息消费者。

```yaml
server:
  port: 8802 # 端口
spring:
  application:
    name: stream-consumer # 应用名称
  rabbitmq:
    host: 192.168.10.101  # 服务器 IP
    port: 5672            # 服务器端口
    username: guest       # 用户名
    password: guest       # 密码
    virtual-host: /       # 虚拟主机地址
```



#### 7.3 代码重构

​		消息生产者 `MyMessageProducer02.java`。

```java
package com.liuk.cloud.service.impl;

import com.liuk.cloud.produce.MySource;
import com.liuk.cloud.produce.MySource02;
import com.liuk.cloud.service.MyMessageProvider02;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.messaging.support.MessageBuilder;

import java.util.UUID;

@EnableBinding(MySource02.class)
public class MyMessageProviderImpl02 implements MyMessageProvider02 {
    @Autowired
    private MySource02 mySource02;

    @Override
    public String send() {
        String message = "MY02-" + UUID.randomUUID().toString();
        boolean sendFlag = mySource02.myOutput().send(MessageBuilder.withPayload(message).build());
        System.out.println("message:" + message);
        return message;
    }
}
```

　　

　　消息消费者 `MyMessageConsumer02.java`。

```java
package com.liuk.cloud.consumer;

import com.liuk.cloud.mychannel.MySink02;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.stereotype.Component;

/**
 * 消息消费者
 */
@Component
@EnableBinding(MySink02.class)
public class MyMessageConsumer02 {
    /**
     * 接收消息
     */
    @StreamListener(MySink02.MY_INPUT)
    public void receive(String message) {
        System.out.println("message = " + message);
    }
}
```





#### 7.4 测试

![image-20200913204139275](https://gitee.com/liukai830/picgo/raw/master/image-20200913204139275.png)

![image-20200913204225757](https://gitee.com/liukai830/picgo/raw/master/image-20200913204225757.png)



##### 访问

启动消息消费者，运行测试，消息消费者控制台打印结果如下：

```
接收到消息 -----> 8802: MY02-84f7726d-45e8-4150-be96-3fa9ca542261
接收到消息 -----> 8802: MY02-752d754f-5775-4d1a-9d80-9e7d68dcc31f
接收到消息 -----> 8802: MY02-e4915c72-7f75-4bc6-9b2d-c50927417e32
接收到消息 -----> 8802: MY02-3c5beb22-ebb4-4548-82d4-164a4d32ab0b
接收到消息 -----> 8802: MY02-558179c8-931d-430b-8672-e9b7eebc446d
```

RabbitMQ 界面如下：

![image-20200913204318721](https://gitee.com/liukai830/picgo/raw/master/image-20200913204318721.png)





## 八、短信邮件发送案例

　　一个消息驱动微服务应用可以既是消息生产者又是消息消费者。接下来模拟一个短信邮件发送的消息处理过程：

- 原始消息发送至 `source.message` 交换机；
- 消息驱动微服务应用通过 `source.message` 交换机接收原始消息，经过处理分别发送至 `sms.message` 和 `email.message` 交换机；
- 消息驱动微服务应用通过 `sms.message` 和 `email.message` 交换机接收处理后的消息并发送短信和邮件。



#### 8.1 创建消息通道

　　发送原始消息，接收处理后的消息并发送短信和邮件的消息驱动微服务应用。

```java
package com.liuk.cloud.mychannel;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.SubscribableChannel;

/**
 * 自定义消息通道
 */
public interface MyProcessor {

    String SOURCE_MESSAGE = "source.message";
    String SMS_MESSAGE = "sms.message";
    String EMAIL_MESSAGE = "email.message";

    @Output(SOURCE_MESSAGE)
    MessageChannel sourceOutput();

    @Input(SMS_MESSAGE)
    SubscribableChannel smsInput();

    @Input(EMAIL_MESSAGE)
    SubscribableChannel emailInput();

}
```

　　

　　接收原始消息，经过处理分别发送短信和邮箱的消息驱动微服务应用。

```java
package com.liuk.cloud.mychannel;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.SubscribableChannel;

/**
 * 自定义消息通道
 */
public interface MyProcessor {
    String SOURCE_MESSAGE = "source.message";
    String SMS_MESSAGE = "sms.message";
    String EMAIL_MESSAGE = "email.message";
  
    @Input(SOURCE_MESSAGE)
    MessageChannel sourceOutput();
  
    @Output(SMS_MESSAGE)
    SubscribableChannel smsOutput();
  
    @Output(EMAIL_MESSAGE)
    SubscribableChannel emailOutput();
}
```



#### 8.2 配置文件

　　约定大于配置，配置文件只修改端口和应用名称即可，其他配置一致。

```yaml
spring:
  application:
    name: stream-producer # 应用名称
  rabbitmq:
    host: localhost  # 服务器 IP
    port: 5672            # 服务器端口
    username: guest       # 用户名
    password: guest       # 密码
    virtual-host: /       # 虚拟主机地址
```

　　

```yaml
spring:
  application:
    name: stream-consumer # 应用名称
  rabbitmq:
    host: localhost  # 服务器 IP
    port: 5672            # 服务器端口
    username: guest       # 用户名
    password: guest       # 密码
    virtual-host: /       # 虚拟主机地址
```



#### 8.3 消息驱动微服务A(8801)

##### 发送消息

发送原始消息 `10086|10086@email.com` 至 `source.message` 交换机。

```java
package com.liuk.cloud.service.impl;

import com.liuk.cloud.produce.MyProcessor;
import com.liuk.cloud.service.SourceMessageProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.integration.support.MessageBuilder;


@EnableBinding(MyProcessor.class)
public class SourceMessageProducerImpl implements SourceMessageProducer {
    @Autowired
    private MyProcessor myProcessor;

    @Override
    public void send(String sourceMessage) {
        myProcessor.sourceOutput().send(MessageBuilder.withPayload(sourceMessage).build());
        System.out.println("原始消息发送成功，原始消息为：" + sourceMessage);
    }
}
```



##### 接收消息

接收处理后的消息并发送短信和邮件。

```java
package com.liuk.cloud.controller;


import com.liuk.cloud.produce.MyProcessor;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;

@EnableBinding(MyProcessor.class)
public class SmsAndEmailMessageConsumer {
    /**
     * 接收消息 电话号码
     *
     * @param phoneNum
     */
    @StreamListener(MyProcessor.SMS_MESSAGE)
    public void receiveSms(String phoneNum) {
        System.out.println("电话号码为：" +  phoneNum + "，调用短信发送服务，发送短信...");
    }

    /**
     * 接收消息 邮箱地址
     *
     * @param emailAddress
     */
    @StreamListener(MyProcessor.EMAIL_MESSAGE)
    public void receiveEmail(String emailAddress) {
        System.out.println("邮箱地址为：" + emailAddress + "，调用邮件发送服务，发送邮件...");
    }
}
```





#### 8.4 消息驱动微服务B(8802)

##### 接收消息

接收原始消息 `10086|10086@email.com` 处理后并发送至 `sms.message` 和 `email.message` 交换机。

```java
package com.liuk.cloud.controller;

import com.liuk.cloud.consumer.MyProcessor;
import com.liuk.cloud.service.SmsAndEmailMessageProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;

@EnableBinding(MyProcessor.class)
public class SourceMessageConsumer {
    @Autowired
    private SmsAndEmailMessageProducer smsAndEmailMessageProducer;

    /**
     * 接收原始消息，处理后并发送
     *
     * @param sourceMessage
     */
    @StreamListener(MyProcessor.SOURCE_MESSAGE)
    public void receive(String sourceMessage) {
        System.out.println("原始消息接收成功，原始消息为：" + sourceMessage);
        // 发送消息 电话号码
        smsAndEmailMessageProducer.sendSms(sourceMessage.split("\\|")[0]);
        // 发送消息 邮箱地址
        smsAndEmailMessageProducer.sendEmail(sourceMessage.split("\\|")[1]);
    }
}
```



##### 发送消息

发送电话号码 `10086` 和邮箱地址 `10086@email.com` 至 `sms.message` 和 `email.message` 交换机。

```java
package com.liuk.cloud.service.impl;

import com.liuk.cloud.consumer.MyProcessor;
import com.liuk.cloud.service.SmsAndEmailMessageProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.messaging.support.MessageBuilder;

@EnableBinding(MyProcessor.class)
public class SmsAndEmailMessageProducerImpl implements SmsAndEmailMessageProducer {
    @Autowired
    private MyProcessor myProcessor;

    @Override
    public void sendSms(String smsMessage) {
        myProcessor.smsOutput().send(MessageBuilder.withPayload(smsMessage).build());
        System.out.println("电话号码消息发送成功，消息为：" + smsMessage);
    }

    @Override
    public void sendEmail(String emailMessage) {
        myProcessor.emailOutput().send(MessageBuilder.withPayload(emailMessage).build());
        System.out.println("邮箱地址消息发送成功，消息为：" + emailMessage);
    }
}
```



#### 8.5 测试

```java
@GetMapping("/send4")
public String send4() {
    sourceMessageProducer.send("10086|10086@email.com");
    return "发送成功！" + UUID.randomUUID().toString();
}
```



##### 访问

消息驱动微服务 A 控制台打印结果如下：

```
原始消息发送成功，原始消息为：10086|10086@email.com
电话号码为：10086，调用短信发送服务，发送短信...
邮箱地址为：10086@email.com，调用邮件发送服务，发送邮件...
```

消息驱动微服务 B 控制台打印结果如下：

```
原始消息接收成功，原始消息为：10086|10086@email.com
电话号码消息发送成功，消息为：10086
邮箱地址消息发送成功，消息为：10086@email.com
```
![image-20200913210923777](https://gitee.com/liukai830/picgo/raw/master/image-20200913210923777.png)

![image-20200913210946491](https://gitee.com/liukai830/picgo/raw/master/image-20200913210946491.png)



RabbitMQ 界面如下：

![image-20200913211209291](https://gitee.com/liukai830/picgo/raw/master/image-20200913211209291.png)

