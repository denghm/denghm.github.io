---
layout: article
title: postgreSQL允许远程接入
modified: 2020-05-13
author: Voyager
toc: false
comments: true
categories: technology
summary: 配置postgreSQL支持远程接入
---
## 环境
Centos  7
postgreSQL 12

## 前置条件
安装完成postgreSQL


## 步骤
1. 修改postgresql.conf
修改两处，一处是打开在所有地址上的侦听端口
```ini
listen_addresses = '*'                  
port = 5432   
```
修改密码的存储方式
```ini
password_encryption = md5               # md5 or scram-sha-256
```
2. 修改pg_hba.conf

增加以下一行

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               md5
```
含义是 允许 address列（0.0.0.0/0）的主机通过type（host）方式以method(md5）认证方式通过user（all, 任意用户）用户连接database（all,任意数据库）

完成1，2步骤后启动postgresql
```
systemctl restart postgresql-12
```


3. 打开防火墙
```shell
firewall-cmd --zone=public --add-port=5432/tcp --permanent
```

4. 设置数据库用户和密码
安装时已经创建用户postgres, 密码未知，需要修改密码
centos root用户切换到postgres用户（因为该用户未设置密码，故不能直接登录)，执行psql
默认采用的是pg_hba.conf中的local方式登录，所以不需要用户名和密码
```sql
alter user postgres with password 'pg202005';
```
注意
**这里一定要加分号，否则psql不出现任何提示就结束了，让人误以为执行成功**。 我在这里耗费了一个小时才发现
**采用md5验证方式，不需要centos存在同名的用户**


此时用户就可以在任何一台机器上远程访问
```shell
psql -h 127.0.0.1 -d postgres -U postgres
```

