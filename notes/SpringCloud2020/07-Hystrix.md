	Hytrix是一个用于分布式系统的<font color="red">延迟</font>和<font color="red">容错</font>的开源库，在分布式系统中，许多不可避免的会调用失败，比如超时、异常等。Hytrix是一个能够保证在一个依赖出错的情况下，<font color="red">不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性</font>。

​	"断路器"本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），<font color="red">向调用方返回一个符合预期的、可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方无法处理的异常</font>，这样就保证了服务调用方的线程不会被长时间、不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



## 一、Hystrix重要概念

#### 服务降级(fallback)

​	服务器忙，请稍后再试，不让客户端等待并返回一个友好提示

触发情况：

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满

#### 服务熔断(break)

​	类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示

一般步骤为：服务的降级 -> 进而熔断 -> 恢复调用链路

#### 服务限流(flowlimit)

​	秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行。



## 二、Hystrix案例

### 1. 构建payment8001-hystrix

#### 1.1 新增pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

#### 1.2 application.yml

```yaml
server:
  port: 8001
spring:
  application:
    name: cloud-payment-hystrix-service
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka/
```

#### 1.3 controller&service

```java
package com.liuk.cloud.service;
import org.springframework.stereotype.Service;
import java.util.concurrent.TimeUnit;
@Service
public class PaymentService {
    public String paymentInfoOK(Integer id) {
        return "线程池: "+Thread.currentThread().getName()+" paymentInfoOK, id: "+id;
    }

    public String paymentInfoTimeout(Integer id) {
        int timeNumber = 3;
        try {
            TimeUnit.SECONDS.sleep(timeNumber);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "线程池: "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id: "+id+"\t"+" 耗时(秒)"+timeNumber;
    }
}
```

```java
package com.liuk.cloud.controller;
import com.liuk.cloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import javax.annotation.Resource;

@RestController
@Slf4j
public class PaymentController {
    @Resource
    private PaymentService paymentService;
    @Value("${server.port}")
    private String serverPort;
    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentService.paymentInfoOK(id);
        log.info("*******result:"+result);
        return result;
    }
    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentService.paymentInfoTimeout(id);
        log.info("*******result:"+result);
        return result;
    }
}
```

#### 1.4 正常测试

​	启动`cloud-eureka-server7001`和`cloud-provider-payment8001-hystrix`，

​	访问http://localhost:8001/payment/hystrix/ok/31和http://localhost:8001/payment/hystrix/timeout/31，

​	上述module均成功



### 2. 高并发测试

#### 2.1 jmeter

​	打开Jmeter，在Test Plan上右键 Add -> Threads -> Thread Group创建测试计划线程组。来20000个并发压死8001，20000个请求都去访问paymentInfo_TimeOut服务。

![image-20200808220259626](https://gitee.com/liukai830/picgo/raw/master/image-20200808220259626.png)

​	保存后，在Thread Group上右键Add -> Sampler -> HTTP Request

![image-20200808221545764](https://gitee.com/liukai830/picgo/raw/master/image-20200808221545764.png)

​	保存后点击开始即可压力测试，测试过程中，再在浏览器中请求http://localhost:8001/payment/hystrix/ok/31

​	压测结论：上面还是服务提供者8001自己测试，假如此时外部的消费者80也来访问，那消费者只能干等，最终导致消费端80不满意，服务端8001直接被拖死。



#### 2.2 cloud-consumer-order80-feign-hystrix

​	创建新的消费者Module：`cloud-consumer-order80-feign-hystrix`

##### （1）pom.xml

```xml
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    …………
</dependencies>
```

##### （2）application.yml

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
      defaultZone: http://eureka7001.com:7001/eureka
```

##### （3）主启动类以及业务代码

```java
/* 主启动类 */
@SpringBootApplication
@EnableFeignClients
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class, args);
    }
}

/* OrderHystrixController */
package com.liuk.cloud.controller;
@RestController
@Slf4j
public class OrderHystrixController {
    @Resource
    private PaymentHystrixService paymentHystrixService;
    @Value("${server.port}")
    private String serverPort;
    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        log.info("*******result:"+result);
        return result;
    }
    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        log.info("*******result:"+result);
        return result;
    }
}

/* PaymentHystrixService */
@Component
@FeignClient(value = "CLOUD-PAYMENT-HYSTRIX-SERVICE")
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

​	2W个线程压8001，消费端80微服务再去访问正常的OK微服务8001地址http://localhost/consumer/payment/hystrix/timeout/31，结果是要么转圈圈等待，要么消费端报超时错误。



#### 2.3 如何解决

- 超时导致服务(8001)变慢（转圈）：	超时不再等待
- 出错（宕机或者程序运行出错）：	出错要有兜底
- 解决
  - 对方服务(8001)超时了，调用者(80)不能一直卡死等待，必须有服务降级
  - 对方服务(8001)宕机了，调用者(80)不能一直卡死等待，必须有服务降级
  - 对方服务(8001)OK，调用者(80)自己出故障或有自我要求（自己的等待时间小于服务提供者），自己处理降级



### 3. 服务降级

#### 3.1 降低配置

​	@HystrixCommand

#### 3.2 8001_fallback

##### （1）在`PaymentService`增加降级处理

​	一旦调用`paymentInfoTimeout`出错后，会调用`@HystrixCommand`中指定的fallbackMethod指定方法。

commandProperties可以设置默认超时时间，默认是1s，这里设置成了2s。

**<font color="red">注：服务降级的方法需要与原方法签名保持一致（参数，返回值），异常被catch不会触发fallback</font>**

```java
public class PaymentService {
    @HystrixCommand(
        fallbackMethod = "paymentInfoTimeoutHandler",
        commandProperties={
          @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000")
        }
    )
    public String paymentInfoTimeout(Integer id) {
        int timeNumber = 5;
        try {
            TimeUnit.SECONDS.sleep(timeNumber);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "线程池: "+Thread.currentThread().getName()+" paymentInfo_TimeOut,id: "+id+"\t"+" 耗时(秒)"+timeNumber;
    }

    public String paymentInfoTimeoutHandler(Integer id) {
        return "线程池: "+Thread.currentThread().getName()+" paymentInfoTimeoutHandler,id: "+id+"\t"+"8001降级处理";
    }

}
```

##### （2）主启动类激活Hystrix

​	在主启动类上增加注解`@EnableCircuitBreaker`即可激活HystrixCommand。



#### 3.3 80_fallback

##### （1）主启动类激活Hystrix

在主启动类上增加注解`@EnableHystrix即可激活HystrixCommand。

#####（2）配置文件激活Hystrix

```yaml
feign:
  hystrix:
    enabled: true #如果处理自身的容错就开启。开启方式与生产端不一样。
```

##### （3）业务类

​	与8001_fallback处理方式一样，但是这里修改的是**<font color="red">controller</font>**

```java
public class OrderHystrixController {
    @Resource
    private PaymentHystrixService paymentHystrixService;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(
        fallbackMethod = "paymentInfoTimeoutHandler",
        commandProperties={
          @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
        }
    )
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        log.info("*******result:"+result);
        return result;
    }
    public String paymentInfoTimeoutHandler(Integer id) {
        return "线程池: "+Thread.currentThread().getName()+" paymentInfoTimeoutHandler,id: "+id+"\t"+"80降级处理";
    }
}
```



### 4. 服务降级优雅处理

上述服务降级处理的缺陷：

- 每个业务方法都要对应一个兜底的fallback，代码膨胀
- fallback与业务逻辑混在一起，代码混乱
- 有些可以统一fallback，统一和自定义需要分离



#### 4.1 defaultProperties

1. 在controller中定义一个全局通用fallback`paymentGlobalFallback`

2. 在需要使用全局通用fallback的方法上使用注解`@HystrixCommand`
3. 在使用自定义fallback的方法上使用注解`@HystrixCommand(fallbackMethod = "paymentInfoTimeoutHandler")`

```java
@RestController
@Slf4j
@DefaultProperties(defaultFallback = "paymentGlobalFallback")
public class OrderHystrixController {
    @Resource
    private PaymentHystrixService paymentHystrixService;
    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
//    @HystrixCommand(
//            fallbackMethod = "paymentInfoTimeoutHandler",
//            commandProperties={@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")}
//    )
    @HystrixCommand
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        log.info("*******result:"+result);
        return result;
    }
    public String paymentInfoTimeoutHandler(Integer id) {
        return "线程池: "+Thread.currentThread().getName()+" paymentInfoTimeoutHandler,id: "+id+"\t"+"80降级处理";
    }
		/** 通用的fallback,不可以有参数 */
    public String paymentGlobalFallback() {
        return "Global异常处理信息，请稍后再试,(┬＿┬)";
    }
}

```

#### 4.2 业务逻辑分离

​	新建`paymentFallbackService`实现`PaymentHystrixService`接口，并重写对应方法，重写的方法就是对应的fallback。

​	`PaymentHystrixService`的`@FeignClient`增加fallback属性，指定降级处理的类

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-HYSTRIX-SERVICE", fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfoOK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfoTimeOut(@PathVariable("id") Integer id);
}
```

对每个调用进行fallback处理，因为controller一般不涉及业务逻辑，所以一般都是使用这种方式进行服务降级处理。

```java
@Component
public class PaymentFallbackService implements PaymentHystrixService {
    @Override
    public String paymentInfoOK(Integer id) { return "paymentInfoOK FallBack"; }
    @Override
    public String paymentInfoTimeOut(Integer id) { return "paymentInfoTimeOut FallBack"; }
}
```




### 5. 服务熔断

​	熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。

​	在Spring Cloud框架里，熔断机制是通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到达一定阈值（缺省是<font color="red">5秒内20次</font>调用失败），就会启动熔断机制。熔断机制的注解是`@HystrixCommand`



修改`cloud-provider-payment8001-hystrix`

```java
public class PaymentService {
    /*  服务熔断  */
    @HystrixCommand(fallbackMethod = "paymentCircuitBreakerFallback",commandProperties = {
      @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),	//是否开启断路器
      @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),   //请求次数
      @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),  //时间范围
      @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), //失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(Integer id) {
        if(id < 0) {
            throw new RuntimeException("ID不能小于0");
        }
        String serialNumber = IdUtil.simpleUUID();
        return Thread.currentThread().getName()+"\t"+"调用成功,流水号："+serialNumber;
    }
    public String paymentCircuitBreakerFallback(Integer id) {
        return "ID不能小于0,请稍后再试,id="+id;
    }
}
```



自测

- http://localhost:8001/payment/circuit/31  正确
- http://localhost:8001/payment/circuit/-31 错误

多次请求错误后再请求正确状态，发现返回结果是fallback内容。

![state](https://gitee.com/liukai830/picgo/raw/master/state.png)

熔断状态：

- 熔断打开：请求不再进行调用当前服务，内部设置始终一般为MTTR（平均故障处理时间），当打开时长达到时钟所设定则进入熔断状态
- 熔断关闭：熔断关闭不会对服务进行熔断
- 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断



断路器开启/关闭的条件：<font color="blue">快照时间窗口</font>、<font color="blue">请求总数阈值</font>、<font color="blue">错误百分比阈值</font>

- 当满足一定阈值的时候（默认10秒内20个请求次数）
- 当失败率达到一定的时候（默认10秒内超过50%请求失败）
- 到达以上阈值，断路器将会开启
- 当开启的时候，所有请求都不会进行转发
- 一段时间过后（默认是5秒），这个时间断路器是搬开状态，会让其中一个请求进行转发。如果成功，断路器会关闭；如果失败，继续开启。重复4和5。



断路器打开之后：<font color="blue">再次请求不会调用主逻辑，而是直接调用fallback</font>

![image-20200809161955746](https://gitee.com/liukai830/picgo/raw/master/image-20200809161955746.png)

![image-20200809162013508](https://gitee.com/liukai830/picgo/raw/master/image-20200809162013508.png)

![image-20200809162023801](https://gitee.com/liukai830/picgo/raw/master/image-20200809162023801.png)

![image-20200809162036391](https://gitee.com/liukai830/picgo/raw/master/image-20200809162036391.png)



### 6. 服务限流

​	Spring Cloud Alibaba Sentinel.



## 三、Hytrix工作流程



![7378149-8821119882b0fcec](https://gitee.com/liukai830/picgo/raw/master/7378149-8821119882b0fcec.webp)



### 1. 构造HystrixCommand或HystrixObservableCommand对象

```java
HystrixCommand command = new HystrixCommand(arg1, arg2); 
HystrixObservableCommand command = new HystrixObservableCommand(arg1, arg2);
```



### 2. 执行Command 命令

共有4种执行命令的方法，前2种只支持HystrixCommand ，后2种只支持HystrixObservableCommand

- execute(): 同步阻塞直至从依赖服务返回结果或抛出异常
- queue(): 异步模式，返回Future，Future封装返回的内容
- observe() : 直接订阅Observable ，此对象包含了从依赖服务返回的结果
- toObservable() : 返回Observable 对象，当你订阅他时，它会执行Hystrix命令并返回结果

```cpp
// HystrixCommand.execute(): 实际调用queue()的方法
public R execute() {
  return queue().get();
}

// HystrixCommand.queue(): 实际调用toObservable()的方法
public Future<R> queue() {
  final Future<R> delegate = toObservable().toBlocking().toFuture();
  ....
}

// HystrixObservableCommand.observe():实际调用toObservable()的方法
public Observable<R> observe() {
	....
  final Subscription sourceSubscription = toObservable().subscribe(subject);
	....
}
```

​	通过以上的代码，我们可以知道：第1种是同步阻塞性调用，第2种是异步非阻塞性调用，第3、4种是基于发布-订阅响应式的调用。虽然是4种调用方式，其实际最后都是基于toObservable方法来实现的。



### 3. 判断结束是否有缓存

​	如果请求缓存功能开启，并且请求在缓存命中，那么返回一个Observable，此对象包含请求的结束。



### 4. 判断短路器是否开启

​	在执行命令时，Hystrix 如果发现断路器跳闸，那么hystix会跳到步骤8去执行回退(fallback)逻辑。如果断路器没有跳闸，则继续执行步骤5。



### 5. 判断线程池/队列/信号资源是否满了

​	如果命令关联的线程池和队列（或信号量）满了，则不会执行命令，会跳到步骤8去执行回退(fallback)逻辑。



### 6. 执行HystrixObservableCommand.construct()或HystrixCommand.run()

​	执行HystrixCommand.run()或HystrixObservableCommand.construct()时，如果执行超时或者执行失败，则执行会跳到步骤8去执行回退(fallback)逻辑；如果正常结束，Hystrix 会记录一些日志和监控数据，并返回处理结果。



### 7. Calculate Circuit Health

​	Hystrix向断路器报告成功、失败、拒绝和超时。断路器维护一组计数器来统计执行数据。



### 8. 获取 Fallback逻辑

​	当发生如下情况时，Hystrix会尝试执行回退(fallback)逻辑：
- 在执行时construct() or run() ，跑出异常 (发生在步骤6.)
- 断路器打开时，命令被断路 (发生在步骤4.)
- 当执行命令时，依赖的线程池、队列或信号量满(发生在步骤5.)
- 执行命令超时

​	编写回退(fallback)逻辑时，这个逻辑里最好没有网络调用，只从内存中获取或者只有静态的逻辑，这个逻辑保证不会执行失败。如果非要通过网络去获取Fallback,你需要在使用其他HystrixCommand或HystrixObservableCommand封装请求，并且这个请求必须有fallback逻辑且值没有网络调用，只有静态逻辑。



### 9. Return the Successful Response

​	返回执行结束或者Observable





## 四、HytrixDashboard服务监控

### 1. cloud-consumer-hystrix-dashboard9001

#### pom.xml

```xml
 <!--新增hystrix dashboard-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
 <!--在8001，8002都需要添加监控的依赖包-->
 <dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
 </dependency>
```

####application.yml

```yaml
server:
  port: 9001
```

启动后访问http://localhost:9001/hystrix即可看到成功界面。

![image-20200809174432131](https://gitee.com/liukai830/picgo/raw/master/image-20200809174432131.png)



### 2. 监控8001

![image-20200809180541285](https://gitee.com/liukai830/picgo/raw/master/image-20200809180541285.png)