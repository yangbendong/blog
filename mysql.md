##卸载系统自带的Mariadb
[root@hdp265dnsnfs ~]# rpm -qa|grep mariadb
mariadb-libs-5.5.44-2.el7.centos.x86_64
[root@hdp265dnsnfs ~]# rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64

##删除etc目录下的my.cnf文件

[root@hdp265dnsnfs ~]# rm /etc/my.cnf
rm: cannot remove ?etc/my.cnf? No such file or directory

##检查mysql是否存在
[root@hdp265dnsnfs ~]# rpm -qa | grep mysql
[root@hdp265dnsnfs ~]#

##检查mysql组和用户是否存在，如无创建
[root@hdp265dnsnfs ~]# cat /etc/group | grep mysql
[root@hdp265dnsnfs ~]#  cat /etc/passwd | grep mysql

##创建mysql用户组
[root@hdp265dnsnfs ~]# groupadd mysql
##创建一个用户名为mysql的用户并加入mysql用户组
[root@hdp265dnsnfs ~]# useradd -g mysql mysql
##制定password 为111111
[root@hdp265dnsnfs ~]# passwd mysql
Changing password for user mysql.
New password:
BAD PASSWORD: The password is a palindrome
Retype new password:
passwd: all authentication tokens updated successfully.

##安装到/usr/local
[root@hdp265dnsnfs var]# tar -zxvf mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz
[root@hdp265dnsnfs var]# mv mysql-5.7.18-linux-glibc2.5-x86_64/ mysql57

##更改所属的组和用户
[root@hdp265dnsnfs var]# chown -R mysql mysql57/
[root@hdp265dnsnfs var]# chgrp -R mysql mysql57/
[root@hdp265dnsnfs var]# cd mysql57/

[root@hdp265dnsnfs mysql57]# mkdir data

[root@hdp265dnsnfs mysql57]# chown -R mysql:mysql data


从mysql5.7开始不会自动生成my.cnf文件，所以需要手动创建。
### 在etc下新建my.cnf
```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[mysqld]
skip-name-resolve
#设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=/usr/local/mysql57
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql57/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=160M
sql_mode = "STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
```

### 安装和初始化
```
bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql57/ --datadir=/usr/local/mysql57/data/
2017-04-17 17:40:02 [WARNING] mysql_install_db is deprecated. Please consider switching to mysqld --initialize
2017-04-17 17:40:05 [WARNING] The bootstrap log isn't empty:
2017-04-17 17:40:05 [WARNING] 2017-04-17T09:40:02.728710Z 0 [Warning] --bootstrap is deprecated. Please consider using --initialize instead
2017-04-17T09:40:02.729161Z 0 [Warning] Changed limits: max_open_files: 1024 (requested 5000)
2017-04-17T09:40:02.729167Z 0 [Warning] Changed limits: table_open_cache: 407 (requested 2000)
```
**如果是mysql8则**
bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql57/ --datadir=/usr/local/mysql57/data/

```
[root@hdp265dnsnfs mysql57]# cp ./support-files/mysql.server /etc/init.d/mysqld
[root@hdp265dnsnfs mysql57]# chmod 644 /etc/my.cnf
[root@hdp265dnsnfs mysql57]# chmod +x /etc/init.d/mysqld
```
**注意，如果my.cnf的权限太大，启动mysql的时候会报错**
```
[root@localhost local]# service mysqld start
my_print_defaults: [Warning] World-writable config file '/etc/my.cnf' is ignored.
/etc/init.d/mysqld: line 259: cd: /usr/local/mysql: No such file or directory
Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
```
```
[root@hdp265dnsnfs mysql57]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS!
Starting MySQL. SUCCESS!

# 设置开机启动

[root@hdp265dnsnfs mysql57]# chkconfig --level 35 mysqld on
[root@hdp265dnsnfs mysql57]# chkconfig --list mysqld

[root@hdp265dnsnfs mysql57]# chmod +x /etc/rc.d/init.d/mysqld
[root@hdp265dnsnfs mysql57]# chkconfig --add mysqld
[root@hdp265dnsnfs mysql57]# chkconfig --list mysqld
[root@hdp265dnsnfs mysql57]# service mysqld status
 SUCCESS! MySQL running (4475)
```

## 配置文件
修改/etc/profile
添加
```
export PATH=$PATH:/usr/local/mysql57/bin
```
使生效
```
source /etc/profile
```

# 获取初始密码
```
root@hdp265dnsnfs bin]# cat /root/.mysql_secret
# Password set for user 'root@localhost' at 2017-04-17 17:40:02
_pB*3VZl5T<6
```

# 登录以及修改密码
```
[root@hdp265dnsnfs bin]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.18

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set PASSWORD = PASSWORD('111111');
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```
**如果是mysql8，则**
alter user  'root'@'localhost' identified by 'your_password';
# 添加远程访问权限
```
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select host,user from user;
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | root      |
| localhost | mysql.sys |
+-----------+-----------+
rows in set (0.00 sec)


create user 'xxx'@'%' identified by '123';  这里 @‘%’ 表示在任何主机都可以登录
```

# 重启生效
```
/bin/systemctl restart  mysql.service
# 或者
service mysqld restart

[root@hdp265dnsnfs bin]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS!
Starting MySQL. SUCCESS!
```

# 为了在任何目录下可以登录mysql
```
ln -s /usr/local/mysql57/bin/mysql   /usr/bin/mysql
```

#mysql备份
以一般数据库备份时间会选择在访问量很小的半夜，一星期完全备份一次，每天增量备份一次。
这样既不会损耗太大的服务器性能，又可以确保数据安全，通过脚本定时执行的方法来完成夜间自动备份。  
```
mkdir -p /home/mysql/backup/daily
cd /home
# 把路径 /home/mysql的用户和组改成mysql
chown mysql.mysql mysql/
# 然后在/etc/my.cnf文件中加入以下代码：
log-bin = "/home/mysql/logbin.log"
binlog-format = ROW
log-bin-index = "/home/mysql/logindex"
binlog_cache_size=32m
max_binlog_cache_size=512m
max_binlog_size=512m
server-id=1
# 重启mysql
service mysqld restart
```
## 增量备份脚本
增量备份的代码
```
cd /home/mysql
vim binlogbak.sh
# 然后写入以下代码（要改的地方就是：mysqladmin -uroot -proot123 flush-logs;-u+链接数据库的用户 -p+密码 ）：
#!/bin/bash
export LANG=en_US.UTF-8
BakDir=/home/mysql/backup/daily
BinDir=/home/mysql
LogFile=/home/mysql/backup/binlog.log
BinFile=/home/mysql/logindex.index
mysqladmin -uroot -proot123 flush-logs
#这个是用于产生新的mysql-bin.00000*文件
Counter=`wc -l $BinFile |awk '{print $1}'`
NextNum=0
#这个for循环用于比对$Counter,$NextNum这两个值来确定文件是不是存在或最新的。
for file in `cat $BinFile`
do
    base=`basename $file`
#basename用于截取mysql-bin.00000*文件名，去掉./mysql-bin.000005前面的./
    NextNum=`expr $NextNum + 1`
    if [ $NextNum -eq $Counter ]
    then
        echo $base skip! >> $LogFile
    else
        dest=$BakDir/$base
        if(test -e $dest)
        #test -e用于检测目标文件是否存在，存在就写exist!到$LogFile去。
        then
            echo $base exist! >> $LogFile
        else
            cp $BinDir/$base $BakDir
            echo $base copying >> $LogFile
        fi
    fi
done
echo `date +"%Y年%m月%d日 %H:%M:%S"` Bakup succ! >> $LogFile
```
赋予binlogbak.sh执行权限：
```
chmod a+x /home/mysql/binlogbak.sh
```
加入mysqladmin、mysqldump的软连接,/usr/local/mysql57为安装目录
加软连接是因为定时任务中sh会报错  找不到mysqladmin 
```
ln -s /usr/local/mysql57/bin/mysqladmin /usr/bin
ln -s /usr/local/mysql57/bin/mysqldump /usr/bin
```

## 全备份
```
vim databak.sh
#加入以下代码
#要改的地方mysqldump -uroot -proot123 --all-databases --flush-logs --delete-master-logs --single-transaction > $DumpFile；
#!/bin/bash

export LANG=en_US.UTF-8
BakDir=/home/mysql/backup
LogFile=/home/mysql/backup/bak.log
Date=`date +%Y%m%d`
Begin=`date +"%Y年%m月%d日 %H:%M:%S"`

cd $BakDir
DumpFile=$Date.sql
GZDumpFile=$Date.sql.tgz
mysqldump -uroot -proot123 --all-databases --flush-logs --delete-master-logs --single-transaction > $DumpFile
tar -czvf $GZDumpFile $DumpFile
rm $DumpFile

count=$(ls -l *.tgz |wc -l)
if [ $count -ge 5 ]
then
file=$(ls -l *.tgz |awk '{print $9}'|awk 'NR==1')
rm -f $file
fi

Last=`date +"%Y年%m月%d日 %H:%M:%S"`
echo 开始:$Begin 结束:$Last $GZDumpFile succ >> $LogFile
cd $BakDir/daily
rm -f *
```
赋予databak.sh 执行权限：
```
chmod a+x /home/mysql/databak.sh
```
可以使用crontab进行定时备份（略）

## 查看恢复备份
数据有“增删改”操作，增量备份会将操作记录在在logbin.00000*中，
此时先查看logbin.00000*中的记录，登录数据库用如下命令查看：
```
mysql -uroot -p 
# 输入密码
show binlog events in 'logbin.000009';
# 即可得到如下所示：POS点区间的操作就是对数据库的操作，有多种方法恢复，可以按时间，按日志区间（POS点）恢复
mysql> show binlog events in 'logbin.000014';
+---------------+------+----------------+-----------+-------------+---------------------------------------+
| Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                  |
+---------------+------+----------------+-----------+-------------+---------------------------------------+
| logbin.000014 |    4 | Format_desc    |         1 |         123 | Server ver: 5.7.20-log, Binlog ver: 4 |
| logbin.000014 |  123 | Previous_gtids |         1 |         154 |                                       |
| logbin.000014 |  154 | Anonymous_Gtid |         1 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
| logbin.000014 |  219 | Query          |         1 |         297 | BEGIN                                 |
| logbin.000014 |  297 | Table_map      |         1 |         355 | table_id: 221 (backuptest.test)       |
| logbin.000014 |  355 | Update_rows    |         1 |         428 | table_id: 221 flags: STMT_END_F       |
| logbin.000014 |  428 | Xid            |         1 |         459 | COMMIT /* xid=470 */                  |
| logbin.000014 |  459 | Anonymous_Gtid |         1 |         524 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'  |
| logbin.000014 |  524 | Query          |         1 |         602 | BEGIN                                 |
| logbin.000014 |  602 | Table_map      |         1 |         660 | table_id: 221 (backuptest.test)       |
| logbin.000014 |  660 | Delete_rows    |         1 |         717 | table_id: 221 flags: STMT_END_F       |
| logbin.000014 |  717 | Xid            |         1 |         748 | COMMIT /* xid=472 */  
```
恢复
我们直接在这个时候把mysql的日志记录功能关闭，不记录恢复时的日志、
先恢复全备份，再是增量备份
```
vim /etc/my.cnf
# 注释掉添加的记录日志部分
#log-bin = "/home/mysql/logbin.log"
#binlog-format = ROW
#log-bin-index = "/home/mysql/logindex"
#binlog_cache_size=32m
#max_binlog_cache_size=512m
#max_binlog_size=512m
#server-id=1
# 重启mysql服务：
service mysqld restart
```
此时就可以开始恢复了。先恢复最近一次的完整备份,切换到mysql目录下，解压完整备份：
```
cd /home/mysql/backup/  
tar zxvf /home/mysql/backup/20180726.sql.tgz -C /home/mysql/backup  
# 解压得到一个sql文件，输入以下命令然后要求输入数据库密码：  
mysql -uroot -p -v < 20180726.sql  
# 刷新数据库会发现test数据库又回来了，但是还没有后面插入的数据。
# 接下来就要恢复增量备份，结合上面的日志记录，要恢复到最后一条记录插入并且提交的时候，可以用717这个POS点作为结束点，命令如下：  
/usr/local/mysql57/bin/mysqlbinlog --stop-position=717 --database=test /home/mysql/logbin.000013 | mysql -uroot -pmysql123 -v test

```
其实这个增量备份的恢复。就是把这些“增删改”操作重新运行一遍，所以如果有多天的增量备份。则需要在恢复最后一次完整的备份之后，
按时间顺序，一次恢复每一个增量备份的操作，才能得到删除前的完整数据库。