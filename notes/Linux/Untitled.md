固定虚拟机IP：

进入目录：cd /etc/sysconfig/network-scripts/

编辑文件：vi ifcfg-eth33

​	（1）修改BOOTPROTO=static

​	（2）增加APADDR=192.168.9.xxx