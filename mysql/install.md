## 机器准备

操作系统：CentOS 7.8  amd64 

内存：2G 

CPU：2core

| HOST    | IP            | 说明    |
| ------- | ------------- | ------- |
| master1 | 192.168.1.221 | 主节点1 |
| master2 | 192.168.1.222 | 主节点2 |



此部署方式为双主部署，也就是两台机器互为主从



## 环境初始化

```bash
[root@master1 ~]# yum -y upgrade 
[root@master1 ~]# yum install -y wget libaio
```



## MySQL安装

下载MySQL二进制包

```bash
[root@master1 ~]# cd /usr/local/src
[root@master1 src]# wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.34-linux-glibc2.12-x86_64.tar
```

解压包

```bash
[root@master1 src]# tar xvf mysql-5.7.34-linux-glibc2.12-x86_64.tar
[root@master1 src]# ls -al
-rw-r--r--   1 root root  699447296 Mar 26 14:43 mysql-5.7.34-linux-glibc2.12-x86_64.tar
-rw-r--r--   1 7161 31415 665389778 Mar 26 15:42 mysql-5.7.34-linux-glibc2.12-x86_64.tar.gz
-rw-r--r--   1 7161 31415  34054690 Mar 26 15:40 mysql-test-5.7.34-linux-glibc2.12-x86_64.tar.gz
```

解压出来有两个tar.gz的包，其中带test是支持debug的，我们人使用的是不带test的，继续解压：

```bash
[root@master1 src]# tar -zxvf mysql-5.7.34-linux-glibc2.12-x86_64.tar.gz
[root@master1 src]# mv mysql-5.7.34-linux-glibc2.12-x86_64 /usr/local/mysql-5.7.34
```

创建data目录和用户

```bash
[root@master1 src]# mkdir -p /data/mysql
[root@master1 src]# useradd mysql
```

设置属主

```bash
[root@master1 src]# chown -R /usr/local/mysql-5.7.34
[root@master1 src]# chown -R /data/mysql
```

创建my.cnf，也可以使用/usr/local/mysql-5.7.34/support-files 里的my.cnf

```
[root@master1 src]# vim /usr/local/mysql-5.7.34/my.cnf
```

my.cnf内容如下：

```mysql
[mysql]
port=3306
socket=/tmp/mysql.sock
default-character-set=utf8

[mysqld]
user=nobody 
server_id=11000 
datadir=/data/mysql 
default_storage_engine=InnoDB
character_set_server=utf8
transaction_isolation=REPEATABLE-READ
explicit_defaults_for_timestamp=true
log_bin_trust_function_creators=1
federated
port=3306
socket=/tmp/mysql.sock
back_log=150
max_connections=3000
max_connect_errors=10
skip-name-resolve

# thread buffer
table_open_cache=2048
thread_cache_size=8
thread_stack=192K
query_cache_size=512M
query_cache_limit=2M
sort_buffer_size=8M
join_buffer_size=8M
read_buffer_size=2M
read_rnd_buffer_size=16M
max_allowed_packet=16M
bulk_insert_buffer_size=64M
max_heap_table_size=64M
tmp_table_size=64M
sql_mode="ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

#binlog parameters
binlog_cache_size=1M
max_binlog_cache_size=16M
log-bin=mysql-bin
binlog_format=row
sync_binlog=1
expire_logs_days=7
slow_query_log=1
long_query_time=1
max_binlog_size=128M
#binlog-ignore-db=test
#binlog-ignore-db=performance_schema

#MyISAM parameters
key_buffer_size=32M
myisam_sort_buffer_size=16M
myisam_max_sort_file_size=16M
myisam_repair_threads=1

#Innodb storage engine parameters
innodb_file_format=barracuda
innodb_flush_method=O_DIRECT
innodb_file_per_table=1
innodb_strict_mode=1
innodb_buffer_pool_size=8G 
innodb_buffer_pool_instances=1 
innodb_data_home_dir=/data/mysql
innodb_data_file_path=ibdata1:100M:autoextend
innodb_max_dirty_pages_pct=90 
innodb_lock_wait_timeout=2
innodb_thread_concurrency=16 
innodb_log_buffer_size=8M
innodb_log_group_home_dir=/data/mysql
innodb_log_file_size=64M
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=2
#innodb_undo_tablespaces=16
#innodb_undo_directory
#innodb_undo_logs
#innodb_file_io_threads=4 
#innodb_use_native_aio=0 

#master-slave replication parameters
#enforce-gtid-consistency=true
#gtid-mode=on
master_info_repository="TABLE"
relay_log_info_repository="TABLE"
relay_log_recovery=1
skip_slave_start
log_slave_updates=1
relay_log=mysql-relay-bin
[mysqld_safe]
open-files-limit=8192

[client]
default-character-set= utf8
```



初始化数据库：

```bash
[root@master1 src]# cd /usr/local
[root@master1 local]# mysql-5.7.34/bin/mysqld --defaults-file=/usr/local/mysql-5.7.34/my.cnf --lc_messages_dir=/usr/local/mysql-5.7.34/share/ --initialize
```



设置环境变量

```bash
[root@master1 local]# vi /etc/profile.d/path.sh
```

加入如下内容

```
export PATH=/usr/local/mysql-5.7.34/bin:$PATH
```

如果不生效的话，需要source一下

启动mysql

```bash
[root@master1 local]# mysqld_safe --defaults-file=/usr/local/mysql-5.7.34/my.cnf &
```

注意，这一步之后在终端会把root帐号的密码显示出来，需要记住密码



使用root用户登陆mysql

```bash
[root@master1 local]# mysql -uroot -p
Enter password:
```

输入上一步记录下来的密码

查看用户权限

```mysql
mysql> SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;
```

删匿名用户

```bash
mysql> delete from user where User='';
```

root用户只能本地访问

```mysql
mysql> alter user root@localhost identified by 'passwd';
```



## 主从配置

查看server_id:

```mysql
mysql> show variables like "%server_id%";
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| server_id      | 11000 |
| server_id_bits | 32    |
+----------------+-------+
2 rows in set (0.00 sec)
```

要注意，由于我们部署的是主—主架构，为了避免循环复制binlgo需要保证两台机器的server_id是不一样的，可以在my.cnf文件里去配置server_id参数。

配置复制帐号：

```mysql
mysql> grant replication slave on *.* to 'repluser'@'121.43.234.180' identified by 'abc@123';
```

注意， @后面的ip是准许的ip地址，不是当前机器的ip地址。如果有多个可以使用逗号分割



首先我们将master1配置为主节点，master2配置为从节点

查看master1的位点

```mysql
mysql> show  master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000005 |     3578 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

 其中，File是当前binglog文件，Position是当前位点，记住这两个信息。

然后，在mster2节点上进行如下操作

```mysql
mysql> stop slave;
mysql> change master to \
       master_host='192.168.1.221', \
       master_port=3306, \
       master_user='repluser', \
       master_password='abc@123', \
       master_log_file='mysql-bin.000005', \
       master_log_pos=3578;
       
mysql> start slave;
```

这样，我们就将master1配置成了master2的主节点，然后我们查看从节点的运行状态：

```mysql
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 121.40.65.246
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 3578
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 1170
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 3578
              Relay_Log_Space: 1377
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 11000
                  Master_UUID: 0f1b2a43-a8b9-11eb-847e-00163e11a9fd
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

我们重点关注这几个状态：

```
Relay_Master_Log_File: mysql-bin.000005
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

如果看到是yes，就说明配置成功了



然后我们将master2配置成master1的主节点，过程和上面是一模一样的。



## 验证

我们在master1上执行

```mysql
mysql> create database more;
```

然后我们在master2上查看

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| more               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

看到，more库已经同步到从节点了，说明部署成功了！

