---
title: mysql重置密码
date: 2019-09-18 16:18:55
tags:
typora-root-url: ../../source
categories:
- [sundry]
---

参考：[mysql 官方重置密码](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html)

1. 杀掉mysql进程

```shell
# shell> kill `cat /mysql-data-directory/host_name.pid`
sudo kill `sudo cat /usr/local/mysql/data/localhost.pid`
```

2. 重写init.sql

3. ```sql
   --MySQL 5.7.6 and later:
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
   --MySQL 5.7.5 and earlier:
   --SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');
   ```

   用mysql-init启动mysql
   
   ```shell
   # mysqld --init-file=/initFilePath &
   shell> sudo mysqld --init-file=/Users/lineworks/Documents/utilities/mysql/mysql-init &
   ```
   
   
   