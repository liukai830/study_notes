## what is Ribbon

​	Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**<font color="red">客户端</font>**的**<font color="red">负载均衡</font>**工具。

#### 1、Ribbon vs Nginx

​	`Nginx`是服务端负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由`服务端`实现的。

​	`Ribbon`是本地负载均衡，在调用微服务接口时候，会在注册中心上获取信息服务列表之后缓存到JVM本地，从而在`本地`实现RPC远程服务调用技术。



​	Ribbon在工作时分成两步：

- 选选择EurekaServer，优先选择在同一个区域内负载均衡较少的server。

- 再根据用户指定的策略，从server取到的服务注册列表中选择一个地址。

  其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。



## 一、引入ribbon

​	由于`spring-cloud-starter-netflix-eureka-server`中已包含了`ribbon`，所以不用额外引入。

![image-20200808123517836](https://gitee.com/liukai830/picgo/raw/master/image-20200808123517836.png)



## 二、Ribbon负载均衡

### 1. Ribbon核心组件IRule

![image-20200808125722351](https://gitee.com/liukai830/picgo/raw/master/image-20200808125722351.png)



IRule：根据特定算法从服务列表中选取一个要访问的服务：

- RoundRibbonRule：轮询
- RandomRule：随机
- RetryRule：先按照RoundRobinRule的策略获取服务，如果失败在指定时间内会进行重试，获取可用的服务
- WeightedResponseTimeRule：对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- BestAvailableRule：会先过滤掉多次访问故障而处于断路跳闸状态的服务，然后选择一个并发量小的服务
- AvailablityFilteringRule：先过滤掉故障实例，再选择并发较小的实例
- ZoneAvoidanceRule：默认规则，复合判断server所在区域的性能和server的可用性选择服务器



### 2. 规则替换

​	官方文档明确给出了警告：

​	这个自定义配置类不能放在@ComponentScan所扫描的当前包以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

#### （1）修改`cloud-payment-80`，增加自定义配置类

​	新建`com.liuk.myruleMySelfRule`

```java
package com.liuk.myrule;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```



#### （2）修改主启动类

```java
package com.liuk.cloud;
import com.liuk.myrule.MySelfRule;
@SpringBootApplication
@EnableDiscoveryClient
// 新增Ribbon自定义的配置类注解
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

#### （3）测试

访问http://localhost/consumer/payment/get/1即可看到返回的数据是8001和8002两个微服务随机调用的。



### 3. 轮询算法原理

负载均衡算法：rest接口第几次请求数%服务器集群总数量=实际调用服务器位置下标，每次服务重启后rest接口计数从1开始。

`List<ServiceInstance> instances = discoveryClient,getInstances("CLOUD-PAYMENT-SERCICE");`

如： 

​	List[0] instance = 127.0.0.1:8002
​	List[1] instance = 127.0.0.1:8001

8001+8002组合成为集群，它们攻击2台机器，集群总数为2，按照轮询算法原理：

​	当总请求数为1时：1%2=1 对应下标位置为1，则获得服务地址为127.0.0.1:8001
​	当总请求数为2时：2%2=0 对应下标位置为0，则获得服务地址为127.0.0.1:8002
​	当总请求数为3时：3%2=1 对应下标位置为1，则获得服务地址为127.0.0.1:8001
​	当总请求数为4时：4%2=0 对应下标位置为0，则获得服务地址为127.0.0.1:8002
​	如此类推……



### 4. 默认轮询`RoundRobinRule`源码

```java
package com.netflix.loadbalancer;
import com.netflix.client.config.IClientConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;
public class RoundRobinRule extends AbstractLoadBalancerRule {

    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }
        Server server = null;
        int count = 0;
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers();
            List<Server> allServers = lb.getAllServers();
            int upCount = reachableServers.size();
            int serverCount = allServers.size();
            if ((upCount == 0) || (serverCount == 0)) {
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }
            int nextServerIndex = incrementAndGetModulo(serverCount);
            server = allServers.get(nextServerIndex);
            if (server == null) {
                /* Transient. */
                Thread.yield();
                continue;
            }
            if (server.isAlive() && (server.isReadyToServe())) {
                return (server);
            }
            // Next.
            server = null;
        }
        if (count >= 10) {
            log.warn("No available alive servers after 10 tries from load balancer: " + lb);
        }
        return server;
    }

    private int incrementAndGetModulo(int modulo) {
        // 自旋锁，CAS
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}

```



### 5. 手写轮询算法

#### （1）先在8001和8002controller增加mapping

```java
@GetMapping(value = "payment/lb")
public String getPaymentLB(){
  return serverPort;
}
```

####（2）`0rder80`去掉配置类中的`@LoadBalanced`注解

```java
package com.liuk.cloud.config;
@Configuration
public class ApplicationContextConfig {
    @Bean
    // @LoadBalanced   // 使用该注解赋予RestTemplate负载均衡的能力(Ribbon提供的客户端)
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

#### （3）自定义LoadBalancer接口以及实现类

```java
package com.liuk.cloud.lb;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;
import java.util.List;
@Component
public interface LoadBalancer {
    ServiceInstance instances(List<ServiceInstance> serviceInstances);
}
```

```java
package com.liuk.cloud.lb;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;
import java.lang.annotation.Annotation;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;
@Component
public class MyLB implements LoadBalancer {
    private AtomicInteger atomicInteger = new AtomicInteger(0);
    public final int getAndIncrement()
    {
        int current;
        int next;
        do {
            current = this.atomicInteger.get();
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;
        }while(!this.atomicInteger.compareAndSet(current,next));
        System.out.println("*****第几次访问，次数next: "+next);
        return next;
    }

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
        int index = getAndIncrement() % serviceInstances.size();
        return serviceInstances.get(index);
    }
}
```

####（4）修改OrderController

```java
package com.liuk.cloud.controller;
@RestController
@Slf4j
public class OrderController {
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
    @Autowired private RestTemplate restTemplate;
    @Autowired private LoadBalancer loadBalancer;
    @Autowired private DiscoveryClient discoveryClient;
  	// 省略其他代码……	
    @GetMapping(value = "consumer/payment/lb")
    public String getPaymentLB() {
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        if(instances == null || instances.size() <= 0) {
            return null;
        }
        ServiceInstance serviceInstance = loadBalancer.instances(instances);
        URI uri = serviceInstance.getUri();
        return restTemplate.getForObject(uri+"/payment/lb",String.class);
    }
}
```

