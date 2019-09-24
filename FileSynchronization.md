#服务器间文件夹实时同步
从服务器A（192.168.0.41）同步到服务器B（192.168.0.40）
###服务器B（192.168.0.40）安装配置过程
#####安装配置rsync
1.红旗操作系统默认安装了rsync，安装在/usr/bin/rsync 使用命令：  
    rsync -v  #查看版本  
    which rsync  #查看安装路径  
也可自行安装新版本
  
2.创建用户与密码认证文件并赋予600权限  
使用root用户进行操作，自定义密码为edu_ctsi(此密码并非登录密码，只是一个 rsync 与 client 服务器进行交互的凭证，可自定义)
创建 rsync_server.password 文件，并写入【用户名：密码】，记住此处的密码！ 
```
mkdir /usr/local/rsync/  
cd /usr/local/rsync/
echo "root:edu_ctsi" >/usr/local/rsync/rsync_server.passwd    
chmod 600 rsync_server.passwd  
``` 

3.创建rsync配置文件  
在目录/usr/local/rsync下创建文件rsync_server.conf，内容为： 
```
uid = root
gid = root
use chroot = no
max connections = 10
strict modes = yes
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[web]
path = /home/edu/attachment
comment = web file
ignore errors
read only = no
write only = no
hosts allow = 192.168.0.41
hosts deny = *
list = false
uid = root
gid = root
auth users = root
secrets file = /usr/local/rsync/rsync_server.passwd
``` 

4.启动rsync服务  
使用上一步的配置文件启动
/usr/bin/rsync  --daemon --config=/usr/local/rsync/rsync_server.conf  

###服务器A（192.168.0.41）安装配置过程
#####安装配置rsync
1.红旗操作系统默认安装了rsync，使用命令： 
```
rsync -v  #查看版本  
which rsync  #查看安装路径,为usr/bin/rsync
``` 

2.创建用户与密码认证文件并赋予600权限  
使用root用户进行操作，自定义密码为edu_ctsi 同上次记住的密码
```
mkdir /usr/local/rsync/  
cd /usr/local/rsync/    
echo "edu_ctsi" >/usr/local/rsync/rsync_client.passwd    
chmod 600 rsync_client.passwd   s
```
 
3.创建rsync配置文件  
在目录/usr/local/rsync下创建文件rsync_client.conf，内容为：  
```
uid = root
gid = root
use chroot = no
max connections = 10
strict modes = yes
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
[web]
path = /home/edu/attachment
comment = web file
ignore errors
read only = no
write only = no
hosts allow = 192.168.0.40
hosts deny = *
list = false
uid = root
gid = root
auth users = root
secrets file = /usr/local/rsync/rsync_client.passwd
```


4.启动rsync服务  
使用上一步的配置文件启动
/usr/bin/rsync  --daemon --config=/usr/local/rsync/rsync_client.conf  

5.安装inotify  
```
cd /usr/src/
tar zxvf inotify-tools-3.14.tar.gz
cd inotify-tools-3.14
./configure --prefix=/usr/local/inotify
make
make install
```

6.创建监控同步脚本
在/home/edu/文件夹下，创建inoyify.sh,内容： 
```
#!/bin/bash     
host=192.168.0.40
src=/home/edu/attachment/
des=web
password=/usr/local/rsync/rsync_client.passwd
user=root
/usr/local/inotify/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src | while read files     
do    
/usr/bin/rsync -vzrtopg --delete --progress --password-file=/usr/local/rsync/rsync_client.passwd $src $user@$host::$des     
echo "${files} was rsynced" >>/tmp/rsync.log 2>&1     
done
``` 
【注】防火墙需要打开873端口,rsync也可指定其他端口  