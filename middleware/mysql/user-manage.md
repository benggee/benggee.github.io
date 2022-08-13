## 用户创建

```mysql
create user username@'host' # mysql5.7版本不支持
create user username@'host' identified by 'passwd'
grant all on *.* to username@'host' identified by 'passwd'
```

### host说明

```
root@'localhost'

root@'%'   

root@'10.0.0.%'

root@'10.0.%.%'

root@'10.%.%.%'

root@'10.0.0.5%'    范围：50-59  

root@'10.0.0.0/255.255.255.0'
```



### 查找用户

```mysql
select * from mysql.user;
```

 

## 修改用户密码

```mysql
@ grant 
grant all on *.* to username@'host' identified by 'passwd'

@ mysqladmin
mysqladmin -uusername -ppasswd password 'new passwd'

@ update
update mysql.user set password=PASSWORD('new passwod') where user='username' and host ='host'
# 5.7版本以后使用此方法
update mysql.user set authentication_string=PASSWORD('new password') where user='username' and host='host' 
# 刷新权限 
flush privileges

@ set 修改当前用户密码
set password=PASSWORD('new passwd')

@ alter 从5.7.6开始建议使用此方法
ALTER USER 'username'@'host' IDENTIFIED BY 'new passwd'
# 如果当前用户是匿名用户
ALTER USER USER() IDENTIFIED BY 'new passwd';

```

## 修改用户名

```mysql
@ update
update mysql.user set user='username1' where user='username'
# 修改完记得grant 
alter user 'username1'@'host' indentified by 'passwd' 

@ rename 
rename user username@'host' to username1@'host'
```

### 删除用户

```mysql
drop user username@'::1'
delete from mysql.user where user='username' and host='::1'
flush privileges
```

## 权限 

```mysql
show grants # 查看权限 
grant select on testdb.* to 'username'@'host'
flush privileges 

grant update(name,math) on dbtest.tabletest to 'username'@'host' 
flush privileges 

grant all on *.* to 'username'@'host' identified by 'passwd' 
flush privileges 
```

### 权限相关表

mysql.user   mysql.db   mysql.table_priv   mysql.columns_priv  mysql.procs_priv 

### 权限回收

```mysql
revoke select on mysql.user from username@host;  
revoke all privileges,grant option from username@'host';
```

### 忘记root密码

```mysql
systemctl stop mysqld 
mysql_safe --skip-grant-tables --skip-networking & 
update mysql.user set password=PASSWORD('new passwd') where user='root' and host='host';
flush privileges 
```

