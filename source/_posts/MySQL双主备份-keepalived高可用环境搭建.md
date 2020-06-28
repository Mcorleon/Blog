---
title: MySQL双主备份+keepalived高可用环境搭建
date: 2019-03-20 16:26:11
tags: MySQL
categories: 分布式
---



使用2台服务器搭建一个简易、高可用的分布式MySQL集群

<!-- more -->

## 环境

OS：Ubuntu16.04

Database：MySQL 5.7.11

MASTER：192.168.75.129

BACKUP：192.168.75.131

测试机（可选）：192.168.75.1

确保可以互相ping通



## 配置MySQL双主同步

1.找到MySQL配置文件，我的机子是 /etc/mysql/mysql.conf.d/mysqld.cnf ，也可能在/etc/mysql/my.cnf或是其他地方。

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf 
```

修改MASTER配置文件：

```shell
#bind-address        = 127.0.0.1 #允许数据库访问的ip，注释掉或者设置为 ::
server-id            = 1    	 #主机标示，整数
log_bin              = /var/log/mysql/mysql-bin.log   #确保此文件可写，开启bin-log
read-only            =0  		#主机，读写都可以
binlog-do-db         =test   	#指定mysql的binlog日志记录哪个db,多个写多行
binlog-ignore-db     =mysql 	#不需要备份的数据库，多个写多行
auto-increment-offset   = 2		# 设置自增长起始
auto-increment-increment = 2    # 增长起始设置为2

```

修改BACKUP配置文件：

```shell
#bind-address        = 127.0.0.1 #允许数据库访问的ip，注释掉或者设置为 ::
server-id            = 2    	 #主机标示，整数
log_bin              = /var/log/mysql/mysql-bin.log   #确保此文件可写，开启bin-log
read-only            =0  		#主机，读写都可以
binlog-do-db         =test   	#指定mysql的binlog日志记录哪个db,多个写多行
binlog-ignore-db     =mysql 	#不需要备份的数据库，多个写多行
auto-increment-offset   = 2		# 设置自增长起始
auto-increment-increment = 2    # 增长起始设置为2
```

修改后均重启服务 :

```bash
sudo service mysqld restart
```

2.重启服务后继续配置。在MASTER的MySQL为BACKUP创建账号并赋予权限：

```bash
mysql -uroot -p
mysql> GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'backup'@'192.168.75.131' IDENTIFIED BY '123456';
mysql> flush privileges;
```

查看MASTER的binlog日志信息，记下File和Position字段的值：

```mysql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      614 | test         | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+

```

在BACKUP的MySQL执行命令，设置同步的主机为MASTER：

```mysql
mysql> change master to master_host='192.168.75.129',master_user='backup',master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=614;
```

然后开启同步

```mysql
mysql> start slave
```

查看slave信息

```mysql
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.75.129
                  Master_User: backup
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 614
               Relay_Log_File: slave1-relay-bin.000010
                Relay_Log_Pos: 827
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: test
          Replicate_Ignore_DB: mysql


```

观察到：

```
  Slave_IO_Running: Yes
  Slave_SQL_Running: Yes
```

说明主从同步正常运行。

3.同样地，在MASTER上将BACKUP设置为同步的master，也就是进行2中的操作，不赘述。

4.到此为止，双主复制搭建完毕。可自行进行测试，无论MASTER还是BACKUP进行数据的增减，另一台会同步执行操作。



## 配置keepalived

安装keepalived及其依赖

```shell
sudo apt-get install libssl-dev openssl libpopt-dev
sudo apt-get install keepalived
```

修改系统网络配置，打开系统IP转发

```shell
sudo vim /etc/sysctl.conf
 
#net.ipv4.ip_forward=1  #将此处注释打开
net.ipv4.ip_nonlocal_bind=1 #尾行添加这个字段
```

使配置生效

```shell
sudo sysctl -p
```

配置keepalived

```shell
sudo vim /etc/keepalived/keepalived.conf


! Configuration File for keepalived
global_defs {
  router_id LVS_DEVEL_SERVER #主备机路由ID
}

vrrp_instance VI_SERVER {
  state MASTER               # 主机服务器模式，备机设为BACKUP
  interface ens33            # 主机监控网卡实例
  virtual_router_id 51       # VRRP组名，主备机设置必须完全一致
  priority 150               # 优先级(1-254)，主机设置必须比备机高，备机可设为90
  authentication {           # 认证信息，主备机必须完全一致
    auth_type PASS
    auth_pass 1111
  }
  virtual_ipaddress {        # 虚拟IP地址，主备机必须完全一致
    192.168.75.200/24    	 # 注意配置子网掩码
  }

```

网卡和子网掩码可以用ip a命令查看：

![](https://s2.ax1x.com/2019/05/31/VlFfr8.png)

启动服务

```shell
service keepalived start
```



## 高可用测试

MASTER和BACKUP都进行以上的安装配置后，环境搭建就完成了。两台主机可以同步MySQL数据，通过keepalived虚拟出VIP 192.168.75.200来提供访问，192.168.75.200优先指向MASTER，一旦挂了自动切换成BACKUP，用户无感知。

使用测试机192.168.75.1进行测试。测试机OS为windows10

1.运行Navicat通过VIP连接虚拟机的MySQL

![](https://s2.ax1x.com/2019/05/31/VlFza4.png)

2.连接成功，可以查询到数据

![](https://s2.ax1x.com/2019/05/31/VlkTeO.png)

3.在MASTER上执行ip a 发现VIP在MASTER上，BACKUP没看到VIP

![](https://s2.ax1x.com/2019/05/31/VlA8pR.md.png)

4.停止MASTER的keepalived服务

```shell
service keepalived stop
```

5.发现测试机上依旧可以连接到MySQL，此时在BACKUP执行ip a可以看到VIP，说明VIP已经自动转移给备份机

6.继续关闭BACKUP 的keepalived服务，此时测试机当然就连不上MySQL啦  (*￣︶￣)  VIP 192.168.75.200不存在了。



至此，一个高可用的MySQL双主备份环境搭建完毕。