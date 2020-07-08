# OS-Linux-
linux相关电子书
# <a href="https://man.linuxde.net/download/CentOS_6_5">CentOS6.5下载，CentOS 6.5 iso下载</a>
# <a href="http://mirror.nsc.liu.se/centos-store/6.5/isos/x86_64/">CentOS 6.5 iso下载2</a>
# <a href="http://www.linuxdown.net/CentOS/2014/0928/3371.html">CentOS 6.5 iso下载3</a>
# <a href="https://developer.aliyun.com/article/443121">Centos 6.5X64 安装mysql5.6</a>
# <a href="https://github.com/kennycaiguo/OS-Linux-and-more/blob/master/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/centos6.5X64%E5%AE%89%E8%A3%85fastdfs.txt">cnetos6.564位安装fastDFS</a>
# <a href="https://learnku.com/articles/35042">MySQL 安装常见错误</a>
# <a href="https://juejin.im/post/5c088b066fb9a049d4419985">CentOS 7 - 安装MySQL 5.7</a>
# 关于mysql5.7的配置问题
想进入mysql的时候碰到问题

[root@localhost yum.repos.d]# mysql -uroot -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[root@localhost yum.repos.d]# mysql -uroot -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
[root@localhost yum.repos.d]# mysql -uroot 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
 
经过查文档发现这是由于没有在user表中写入用户信息，只能使用不启动授权表的启动方式：
修改my.cnf配置

[root@localhost etc]# vim /etc/my.cnf

# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

skip-grant-tables
 

重启MySQL服务

[root@localhost etc]# systemctl restart mysqld
1
进入数据库，修改user表

[root@localhost etc]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
 
这时候如果输入mysql> update user set password=password("oracle") where user="root";会报错
ERROR 1054 (42S22): Unknown column 'password' in 'field list'
因为这是由于5.7版本user表的字段名做了变更
查询user表的字段：select * from user\G
可知道password字段已经变成了authentication_string字段，对我们的代码稍作修改即可：
mysql> update mysql.user set authentication_string=password('oracle') where user='root' ;
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1
 
刷新系统权限并退出

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> quit
Bye

再次进入MySQL发现还是有问题

[root@localhost etc]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.19

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases
    -> ;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> set password=password('oracle');
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
 
查文档后发现原来MySQL 5.7在安装后会自动启动密码限制进程，密码必须有一定复杂度，即包含大小写和数字，如’MyNewPass4!’，我给出的密码oracle显然不符合标准
由于只是测试环境，我不希望用复杂的密码，所以只能去修改密码限制了：

mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_length=1;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_mixed_case_count=2;
Query OK, 0 rows affected (0.00 sec)
 
这时再去修改密码即可

mysql> alter user 'root'@'localhost' identified by 'oracle';
Query OK, 0 rows affected (0.00 sec)

mysql> quit
Bye
 
此时MySQL数据库正式配置完成
 
# <a href="https://blog.csdn.net/dejunyang/article/details/79836502">Centos 7 安装 vs code</a>
# Centos7 安装dotnet core sdk 3.1:
•	
随笔 - 194  文章 - 0  评论 - 1958
CentOS 7 安装.NET Core 3.1
一.添加dotnet产品Feed
在安装.NET Core之前，您需要注册Microsoft产品Feed。 这只需要做一次。 首先，注册Microsoft签名密钥，然后添加Microsoft产品Feed。
rpm --import https://packages.microsoft.com/keys/microsoft.asc
sh -c 'echo -e "[packages-microsoft-com-prod]\nname=packages-microsoft-com-prod \nbaseurl=https://packages.microsoft.com/yumrepos/microsoft-rhel7.3-prod\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/dotnetdev.repo'
二.安装 .NET Core SDK
请先从系统中删除任何以前的预览版本的.NET Core，然后再进行下一步。
以下命令更新可用于安装的产品列表，安装.NET Core所需的组件，然后安装.NET Core SDK。
yum update
yum install libunwind libicu
yum install dotnet-sdk-3.1
三.编写代码验证安装
使用命令新建一个控制台应用程序
dotnet new console -o hwapp
cd hwapp
该命令将会新建一个 Program.cs 文件，将会输出“Hello Word”
四.运行程序
dotnet run


# VMware虚拟机不能打开的解决办法
	打开运行命令，
  输入services.msc
  查看VMware Authorization Service服务有没有启动，如果没有，将其启动，必须步骤所有的VMware 相关服务都正常运行
   可是虚拟机还是不能打开
   有可能虚拟机已经损坏了，可能是因为强制关闭造成的。不过虚拟磁盘是好的。解决办法：在Vmware控制台把虚拟机删除，这个不会删除磁盘文件，
   新建虚拟机，选择使用现有虚拟磁盘，其余步骤一样确定
  启动新建的虚拟机，发现能够正常启动了
  
# Centos  卸载软件：
 
yum安装：yum remove xxx
rpm包安装：rpm -e xxx
tar包安装：直接删除文件或make uninstall xxx
# Centos使用 yum history 卸载软件及其依赖
 
yum带有历史记录功能，可以查看过往的事务，重做或回滚这些事务

最重要的是可以连带依赖一并删除

yum history

root@localhost ~]# yum history
已加载插件：fastestmirror
ID     | 登录用户                 | 日期和时间       | 操作           | 变更数 
-------------------------------------------------------------------------------
     3 | root <root>              | 2018-08-17 00:02 | Install        |   30   
     2 | root <root>              | 2018-08-17 00:01 | Install        |    4   
     1 | 系统 <空>                | 2018-08-17 07:39 | Install        |  299   
history list
=============================================================================

在历史中搜索某个软件包是

yum history list Name/ ID

[root@localhost ~]# yum history list nxlog-ce
已加载插件：fastestmirror
ID     | 命令行                   | 日期和时间       | 操作           | 变更数 
-------------------------------------------------------------------------------
     3 | localinstall nxlog-ce-2. | 2018-08-17 00:02 | Install        |   30   
history list
[root@localhost ~]# yum history list 5
已加载插件：fastestmirror
ID     | 命令行                   | 日期和时间       | 操作           | 变更数 
-------------------------------------------------------------------------------
     5 | install vim              | 2018-08-17 00:43 | Install        |   31   
=============================================================================

回滚是

yum history undo ID

[root@localhost ~]# yum history undo 5
已加载插件：fastestmirror
Undoing transaction 5, from Fri Aug 17 00:43:45 2018
*
*
*
事务概要
===========================================================================================
移除  31 软件包
 
安装大小：60 M
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
*
*
*
删除:
  gpm-libs.x86_64 0:1.20.7-5.el7                      perl.x86_64 4:5.16.3-292.el7                 perl-Carp.noarch 0:1.26-244.el7              
  perl-Encode.x86_64 0:2.51-7.el7                     perl-Exporter.noarch 0:5.68-3.el7            perl-File-Path.noarch 0:2.09-2.el7           
  perl-File-Temp.noarch 0:0.23.01-3.el7               perl-Filter.x86_64 0:1.49-3.el7              perl-Getopt-Long.noarch 0:2.40-3.el7         
  perl-HTTP-Tiny.noarch 0:0.033-3.el7                 perl-PathTools.x86_64 0:3.40-5.el7           perl-Pod-Escapes.noarch 1:1.04-292.el7       
  perl-Pod-Perldoc.noarch 0:3.20-4.el7                perl-Pod-Simple.noarch 1:3.28-4.el7          perl-Pod-Usage.noarch 0:1.63-3.el7           
  perl-Scalar-List-Utils.x86_64 0:1.27-248.el7        perl-Socket.x86_64 0:2.010-4.el7             perl-Storable.x86_64 0:2.45-3.el7            
  perl-Text-ParseWords.noarch 0:3.29-4.el7            perl-Time-HiRes.x86_64 4:1.9725-3.el7        perl-Time-Local.noarch 0:1.2300-2.el7        
  perl-constant.noarch 0:1.27-2.el7                   perl-libs.x86_64 4:5.16.3-292.el7            perl-macros.x86_64 4:5.16.3-292.el7          
  perl-parent.noarch 1:0.225-244.el7                  perl-podlators.noarch 0:2.5.1-3.el7          perl-threads.x86_64 0:1.87-4.el7             
  perl-threads-shared.x86_64 0:1.43-6.el7             vim-common.x86_64 2:7.4.160-4.el7            vim-enhanced.x86_64 2:7.4.160-4.el7          
  vim-filesystem.x86_64 2:7.4.160-4.el7              
 
完毕！
 
# CentOS 利用 yum 安装卸载软件常用命令
一、yum安装和卸载软件
有个前提是yum安装的软件包都是rpm格式的。
安装的命令是，yum install ~，yum会查询数据库，有无这一软件包，如果有，则检查其依赖冲突关系，如果没有依赖冲突，那么最好，下载安装;如果有，则会给出提示，询问是否要同时安装依赖，或删除冲突的包，你可以自己作出判断；
删除的命令是，yum remove ~，同安装一样，yum也会查询数据库，给出解决依赖关系的提示。
其中~ 代表软件名
1.用YUM安装软件包命令：yum install ~
2.用YUM删除软件包命令：yum remove ~

回到顶部
二、用yum查询想安装的软件
我们常会碰到这样的情况，想安装一个软件，只知道它和某方面有关，但又不能确切知道它的名字。这时yum的查询功能就起作用了。我们可以用 yum search keyword这样的命令来进行搜索，比如我们要则安装一个Instant Messenger，但又不知到底有哪些，这时不妨用 yum search messenger这样的指令进行搜索，yum会搜索所有可用rpm的描述，列出所有描述中和messeger有关的rpm包，于是我们可能得到 gaim，kopete等等，并从中选择。有时我们还会碰到安装了一个包，但又不知道其用途，我们可以用yum info packagename这个指令来获取信息。
1.使用YUM查找软件包
命令：yum search ~
2.列出所有可安装的软件包
命令：yum list
3.列出所有可更新的软件包
命令：yum list updates
4.列出所有已安装的软件包
命令：yum list installed
5.列出所有已安装但不在Yum Repository 內的软件包
命令：yum list extras
6.列出所指定软件包
命令：yum list ～
7.使用YUM获取软件包信息
命令：yum info ～
8.列出所有软件包的信息
命令：yum info
9.列出所有可更新的软件包信息
命令：yum info updates
10.列出所有已安裝的软件包信息
命令：yum info installed
11.列出所有已安裝但不在Yum Repository 內的软件包信息
命令：yum info extras
12.列出软件包提供哪些文件
命令：yum provides ~

回到顶部
三、清除YUM缓存
yum 会把下载的软件包和header存储在cache中，而不会自动删除。如果我们觉得它们占用了磁盘空间，可以使用yum clean指令进行清除，更精确的用法是yum clean headers清除header，yum clean packages清除下载的rpm包，yum clean all 清除所有。
1.清除缓存目录(/var/cache/yum)下的软件包
命令：yum clean packages
2.清除缓存目录(/var/cache/yum)下的 headers
命令：yum clean headers
3.清除缓存目录(/var/cache/yum)下旧的 headers
命令：yum clean oldheaders
4.清除缓存目录(/var/cache/yum)下的软件包及旧的headers
命令：yum clean, yum clean all (= yum clean packages; yum clean oldheaders)

回到顶部
四、yum命令工具使用举例
yum update 升级系统
yum install ～ 安装指定软件包
yum update ～ 升级指定软件包
yum remove ～ 卸载指定软件
yum grouplist 查看系统中已经安装的和可用的软件组，可用的可以安装
yum grooupinstall ～安装上一个命令显示的可用的软件组中的一个
yum grooupupdate ～更新指定软件组的软件包
yum grooupremove ～ 卸载指定软件组中的软件包
yum deplist ～ 查询指定软件包的依赖关系
yum list yum\* 列出所有以yum开头的软件包
yum localinstall ～ 从硬盘安装rpm包并使用yum解决依赖

 

转自：http://tech.v01.cn/Linuxchangjianwenti/changyongruanjiananzhuangyucao/2012/0119/70.html 
 
# nginx完整启动脚本
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
#description:   NGINX is an HTTP(S) server,HTTP(S）reverse \
#               proxy and IMAP/POP3 proxy server
#proccessname: nginx
#config:       /etc/nginx/nginx.conf
#config:       /etc/sysconfig/nginx
#pidfile:      /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

#Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/bin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/opt/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/ngix ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs(){
 ##make required directories
 user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
 if [ -n "$user" ];then
    if [ -z "`grep $user /etc/passwd`" ];then
        useradd -M -s /bin/nologin $user
   fi
    options=`$nginx -V 2>&1 |grep 'configure arguments:'`
    for opt in $options;do
        if [ `echo $opt | grep '.*-temp-path'`];then
            value=`echo $opt | cut-d "=" -f 2`
             if [ !-d "$value" ]; then
                  # #echo "creating" $value
                  mkdir -p $value && chown -R $user $value
            fi
         fi
     done
   fi
}
start(){
     [ -x $nginx ] || exit 5
     [ -f $NGINX_CONF_FILE ] || exit 6
     make_dirs
     echo -n $"Starting  $prog: "
     daemon $nginx -c $NGINX_CONF_FILE
     retval=$?
     echo
     [ $retval -eq 0 ] && touch $lockfile
     return $retval
}

stop(){
     echo -n $"Stopping $prog: "
     killproc $prog -QUIT
     retval=$?
     echo
     [ $retval -eq 0 ] && rm -f $lockfile
     return $retval
}
retart(){
      configtest || return $?
      stop
      sleep 1
      start
}      

reload(){
     configtest || return $?
     echo -n $"Reloading $prog: "
     #killproc $ngnix -HUP
     killproc $prog -HUP
     RETVAL=$?
     echo
}

force_reload(){
      restart
}

configtest(){
    $nginx -t -c $NGINX_CONF_FILE
  }

rh_status(){
    status $prog
}

rh_status_q(){
    rh_status >/dev/null 2>&1
}

case "$1" in
         start)
             rh_status_q && exit 0
             $1
             ;;
          stop)
             rh_status_q || exit 0
             $1
             ;;
           restart|configtest)
             $1
             ;;
           reload)
             rh_status_q || exit 7
             $1
             ;;
            force-reload)
               force_reload
               ;;
             status)
                rh_status
                ;;
             condrestart|try-restart)
                 rh_status_q || exit 0
                     ;;
            *)
             
               echo $"Usage: $0 {start|stop|status|restart|condrestart|try- restart|reload|force-reload|configtest}"
           
                    exit 2
esac

 

