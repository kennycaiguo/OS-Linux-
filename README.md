# OS-Linux-
linux相关电子书
# <a href="https://man.linuxde.net/download/CentOS_6_5">CentOS6.5下载，CentOS 6.5 iso下载</a>
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

 

