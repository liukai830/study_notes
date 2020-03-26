

# 一、安装

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
# 先在/usr/local/zookeeper-3.4.14下创建文件夹dataDir和dataLogDir
dataDir=/usr/local/zookeeper-3.4.14
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
```

# 二、注册中心Zookeeper

zookeeper是一个分布式协调工具，可以实现注册中心功能