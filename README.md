# OS-Linux-
linux相关电子书
# <a href="https://man.linuxde.net/download/CentOS_6_5>CentOS6.5下载，CentOS 6.5 iso下载</a>
# VMware虚拟机不能打开的解决办法
	打开运行命令，
  输入services.msc
  查看VMware Authorization Service服务有没有启动，如果没有，将其启动，必须步骤所有的VMware 相关服务都正常运行
   可是虚拟机还是不能打开
   有可能虚拟机已经损坏了，可能是因为强制关闭造成的。不过虚拟磁盘是好的。解决办法：在Vmware控制台把虚拟机删除，这个不会删除磁盘文件，
   新建虚拟机，选择使用现有虚拟磁盘，其余步骤一样确定
  启动新建的虚拟机，发现能够正常启动了
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

 

