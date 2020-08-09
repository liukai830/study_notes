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

​	![20200318202547900](https://gitee.com/liukai830/picgo/raw/master/20200318202547900.png)

Web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化的控制。
​	Predicate就是我们的匹配条件；而Filter就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了。



## 三、Gateway工作流程

<center class="half">
<img src="https://gitee.com/liukai830/picgo/raw/master/无标题.png" width="500"/>
    <img src="https://gitee.com/liukai830/picgo/raw/master/12191355-28ebd217899aa37e.png" width="500"/>
</class>
​	客户端向Spring Cloud Gateway 发出请求，然后在Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway Web Handler。
​	Handler再通过指定的过滤连将请求大宋我实际的服务执行业务，然后返回。过滤器质检用虚线分开是因为过滤器可能会在发送代理请求之前("pre")或之后("post")执行业务逻辑。
​	Filter在"pre"类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等；在"post"类型的过滤器中可以做响应内容、响应头的修改、日志的输出、流量监控等有着非常重要的作用。





## 四、入门配置



## 五、通过微服务名实现动态路由







## 六、Predicate的使用









## 七、Filter的使用

