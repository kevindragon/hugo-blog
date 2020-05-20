---
title: keepalived安装配置
date: 2016-01-07 10:26:28
tags: [keepalived]
---

QQ: 380800878, 微信: kittenll

下载源码包，直接上[官方网站](http://www.keepalived.org/)

解压

    tar -xzvf keepalived-1.2.7.tar.gz

编译安装，没什么特别的，指定一下安装目录即可

    # cd keepalived-1.2.7
    # ./configure --prefix=/usr/local/keepalived-1.2.7
    # make
    # make install

添加软链，升级测试方便

    # cd /usr/local/
    # ln -s keepalived-1.2.7/ keepalived
    # ln -s /usr/local/keepalived/sbin/keepalived /usr/bin/

添加keepalived配置文件

    # mkdir /etc/keepalived
    # touch /etc/keepalived/keepalived.conf

把如下内容添加进到`keepalived.conf`

    ! Configuration File for keepalived

    global_defs {
        router_id nginx-proxy-ha
    }
    
    vrrp_script check_nginx_php {
        script "/usr/local/keepalived/sbin/check_nginx_php.sh"
        interval 5
        weight 2
    }
    
    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 51
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1234
        }
        track_interface {
            eth0
        }
        track_script {
            check_nginx_php
        }
        virtual_ipaddress {
            192.168.1.100
        }
        virtual_ipaddress {
            192.168.1.99
        }
    }

**把对应的IP地址替换成自己内网IP即可**

    # touch /etc/sysconfig/keepalived

添加如下内容

    # Options for keepalived. See `keepalived --help' output and keepalived(8) and
    # keepalived.conf(5) man pages for a list of all options. Here are the most
    # common ones :
    #
    # --vrrp               -P    Only run with VRRP subsystem.
    # --check              -C    Only run with Health-checker subsystem.
    # --dont-release-vrrp  -V    Dont remove VRRP VIPs & VROUTEs on daemon stop.
    # --dont-release-ipvs  -I    Dont remove IPVS topology on daemon stop.
    # --dump-conf          -d    Dump the configuration data.
    # --log-detail         -D    Detailed log messages.
    # --log-facility       -S    0-7 Set local syslog facility (default=LOG_DAEMON)
    #
    
    KEEPALIVED_OPTIONS="-D"

添加如下开机启动的脚本

    # touch /etc/rc.d/init.d/keepalived

内容如下

    #!/bin/sh
    #
    # Startup script for the Keepalived daemon
    #
    # processname: keepalived
    # pidfile: /var/run/keepalived.pid
    # config: /etc/keepalived/keepalived.conf
    # chkconfig: - 21 79
    # description: Start and stop Keepalived
    
    # Source function library
    . /etc/rc.d/init.d/functions
    
    # Source configuration file (we set KEEPALIVED_OPTIONS there)
    . /etc/sysconfig/keepalived
    
    RETVAL=0
    
    prog="keepalived"
    
    start() {
        echo -n $"Starting $prog: "
        daemon keepalived ${KEEPALIVED_OPTIONS}
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && touch /var/lock/subsys/$prog
    }
    
    stop() {
        echo -n $"Stopping $prog: "
        killproc keepalived
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$prog
    }
    
    reload() {
        echo -n $"Reloading $prog: "
        killproc keepalived -1
        RETVAL=$?
        echo
    }
    
    # See how we were called.
    case "$1" in
        start)
            start
            ;;
        stop)
            stop
            ;;
        reload)
            reload
            ;;
        restart)
            stop
            start
            ;;
        condrestart)
            if [ -f /var/lock/subsys/$prog ]; then
                stop
                start
            fi
            ;;
        status)
            status keepalived
            ;;
        *)
            echo "Usage: $0 {start|stop|reload|restart|condrestart|status}"
            exit 1
    esac
    
    exit $RETVAL
