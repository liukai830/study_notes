## 1. kafka安装

#### （1）解压安装包

```shell
[root@kafka-01 local]$ tar -zxvf kafka_2.11-0.11.0.0.tgz
```

####（2）修改解压后的文件名称  

```shell
[root@kafka-01 local]$ mv kafka_2.11-0.11.0.0/ kafka
```

#### （3）在kafka下创建logs文件夹

​	这个文件夹不是日志文件，是存放的操作数据

```shell
[root@kafka-01 kafka]$ mkdir logs
```

#### （4）修改配置文件

```shell
[root@kafka-01 kafka]$ vim config/server.properties
```

```shell
#broker 的全局唯一编号，不能重复
broker.id=0
#删除 topic 功能使能
delete.topic.enable=true
#kafka 运行日志存放的路径
log.dirs=/opt/module/kafka/logs
#配置连接 Zookeeper 集群地址，多个用逗号隔开
zookeeper.connect=192.168.9.119:2181
```

#### （5）配置环境变量  

```shell
[root@kafka-01 kafka]$ vim /etc/profile

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin

[root@kafka-01 kafka]$ source /etc/profile
```

#### （6）分别在kafka-02、kafka-03上安装

#### （7）启动和关闭

```sh
[root@kafka-01 kafka]$ bin/kafka-server-start.sh -daemon config/server.properties
[root@kafka-01 kafka]$ bin/kafka-server-stop.sh
```

#### （8）kafka群起脚

```shell
#!/bin/bash
case $1 in
"start"){
	for i in 192.168.9.121 192.168.9.122 192.168.9.123
	do
		echo "========== $i =========="
		ssh $i "/usr/local/kafka/bin/kafka-server-start.sh -daemon /usr/local/kafka/config/server.properties"
	done
};;
"stop"){
	for i in 192.168.9.121 192.168.9.122 192.168.9.123
	do
		echo "========== $i =========="
		ssh $i "/usr/local/kafka/bin/kafka-server-stop.sh"
	done
};;
```