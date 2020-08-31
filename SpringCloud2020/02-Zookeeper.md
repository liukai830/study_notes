## 一、安装

```shell
cd /usr/local
# 1、下载
yum install wget
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz

# 2、解压
tar zvxf zookeeper-3.4.14.tar.gz 

# 3、进入 zookeeper 的 conf 目录，拷贝 zoo_samle.cfg 为 zoo.cfg
cd zookeeper-3.4.14/conf/
cp zoo_sample.cfg zoo.cfg

# 4、修改 zoo.cfg 的 dataDir 和 dataLogDir。其余配置参考官方文档。
# 先在/usr/local/zookeeper-3.4.14下创建文件夹data和log
dataDir=/usr/local/zookeeper-3.4.14/data
dataLogDir=/usr/local/zookeeper-3.4.14/log

# 5、修改环境变量
sudo vi /etc/profile
# 在一个合适的位置，添加
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.14
export PATH=$PATH:$ZOOKEEPER_HOME/bin 
# 6、使环境变量生效
source /etc/profile
# 启动验证
./zkServer.sh start	# 启动
./zkCli.sh 	# 连接

# 7、开机启动
# 7.1、编辑zookeeper.service文件
vim /usr/lib/systemd/system/zookeeper.service 
# 增加以下内容
[Unit]
Description=zookeeper
After=network.target remote-fs.target nss-lookup.target
[Service]
Type=forking
ExecStart=/usr/local/zookeeper-3.4.14/bin/zkServer.sh start
ExecReload=/usr/local/zookeeper-3.4.14/bin/zkServer.sh restart
ExecStop=/usr/local/zookeeper-3.4.14/bin/zkServer.sh stop
[Install]
WantedBy=multi-user.target
# 7.2 生效
systemctl daemon-reload
# 7.3 改变文件权限
chmod 777 /usr/lib/systemd/system/zookeeper.service
# 7.4 systemctl开机启动zookeeper
systemctl enable /usr/lib/systemd/system/zookeeper.service
# 7.5、查看是否开机启动
systemctl is-enabled zookeeper.service
# 7.6、systemctl取消开机启动redis
systemctl disable zookeeper.service

# 8、开放端口
firewall-cmd --zone=public --add-port=2181/tcp --permanent
firewall-cmd --reload
```



## 二、支付服务注册到zookeeper

### 1、创建`cloud-provider-payment8004`支付服务

#### （1）pom.xml

​	由于spring-cloud-starter-zookeeper-discovery自己带的zk版本与服务器版本(3.4.14)不一致，所以需要将自带的zk排除，引入新的zk包；

​	引入的zk包中的slf4j-log4j12与原有的slf4j产生冲突，所以也需要进行排除。

完整的POM内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.liuk.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8004</artifactId>

    <dependencies>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.liuk.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.9版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
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
</project>
```

#### （2）application.yml

```yaml
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004

#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 118.25.179.80:2181
```



#### （3）验证测试1

![image-20200806204711656](https://gitee.com/liukai830/picgo/raw/master/image-20200806204711656.png)



#### （4）验证测试2

![image-20200806205048416](https://gitee.com/liukai830/picgo/raw/master/image-20200806205048416.png)

获取实例信息json解析后如下：

![image-20200806205150374](https://gitee.com/liukai830/picgo/raw/master/image-20200806205150374.png)

#### （5）永久节点、临时节点？

​	经测试，将payment8004关闭后，zk中的节点会被删除，说明zk使用的是`临时节点`

​	zk是强一致性，保证数据同步最新；而eureka是高可用，不会删除节点。

![image-20200806205714625](https://gitee.com/liukai830/picgo/raw/master/image-20200806205714625.png)



## 三、订单服务注册到zookeeper

#### （1）pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.liuk.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumer-order80-zk</artifactId>

    <dependencies>
        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!--先排除自带的zookeeper3.5.3-->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!--添加zookeeper3.4.9版本-->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.liuk.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--热部署-->
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

</project>
```

#### （2）application.yml

```yaml
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 80

#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-order-service
  cloud:
    zookeeper:
      connect-string: 118.25.179.80:2181
```



#### （3）OrderZkController

```java
package com.liuk.cloud.controller;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderZkController {
    public static final String INVOKE_URL = "http://cloud-provider-payment";
    @Resource
    private RestTemplate restTemplate;
    @GetMapping(value = "/consumer/payment/zk")
    public String paymentInfo() {
        String result = restTemplate.getForObject(INVOKE_URL+"/payment/zk",String.class);
        return result;
    }
}
```



## 四、测试验证

#### （1）zookeeper

`cloud-order-service`和`cloud-provider-payment`都已经注册到zk中：

```shell
[zk: localhost:2181(CONNECTED) 63] ls /services
[cloud-order-service, cloud-provider-payment]
```

#### （2）服务调用

从订单消费者`cloud-order-service`调用提供者`cloud-provider-payment`成功。

![image-20200806212450504](https://gitee.com/liukai830/picgo/raw/master/image-20200806212450504.png)