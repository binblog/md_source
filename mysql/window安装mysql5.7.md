记录一下window下安装mysql-5.7.17-win32的过程。比较简单，简记以备忘

1.进入解压目录(此时还没有data目录)，初始化
```
> bin\mysqld --initialize --user=mysql --console
mysqld: Could not create or access the registry key needed for the MySQL application
to log to the Windows EventLog. Run the application with sufficient
privileges once to create the key, add the key manually, or turn off
logging for that application.
2016-12-17T08:31:29.700216Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2016-12-17T08:31:29.744219Z 0 [ERROR] Cannot open Windows EventLog; check privileges, or start server with --log_syslog=0
2016-12-17T08:31:33.508434Z 0 [Warning] InnoDB: New log files created, LSN=45790
2016-12-17T08:31:34.020463Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2016-12-17T08:31:34.702502Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 3839240f-c433-11e6-a5a7-e31eec489f48.
2016-12-17T08:31:34.776507Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2016-12-17T08:31:34.813509Z 1 [Note] A temporary password is generated for root@localhost: mJmFyTVdA6:>
```
此时当前目录已经生成了data目录，注意输入内容最后有一个root的随机密码`mJmFyTVdA6:>`

2.启动服务
```
> bin\mysqld.exe --user=mysql &
mysqld: Could not create or access the registry key needed for the MySQL application
to log to the Windows EventLog. Run the application with sufficient
privileges once to create the key, add the key manually, or turn off
logging for that application.
```
命令不会返回，mysql服务已经启动

3.修改密码
```
> bin\mysql -uroot -p
Enter password: ************
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.17

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
密码就是初始化时生成的随机密码`mJmFyTVdA6:>`

```
mysql> set password = password('bin');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
密码已经修改为`bin`