# 说明
和Percona moniotr plugin的最大区别就在于该监控支持多端口，并且端口自动发现，不需要配置。
中文的详细说明，参照[链接](https://highdb.com/zabbix-%E5%A4%9A%E7%AB%AF%E5%8F%A3%E7%9B%91%E6%8E%A7-mysql/)。
注意：template 导入之后显示的item和trigger为0，但是绑定host之后就正常。

如果遇到如下启动agent失败
[root@testhost zabbix]# systemctl start zabbix-agent
Job for zabbix-agent.service failed because the control process exited with error code. See "systemctl status zabbix-agent.service" and "journalctl -xe" for details.

执行命令journalctl -xe，根据提示分析就可以。

如果遇到使用zabbix_get在zabbix server上测试失败，根据提示信息进行修复就可以。


# zabbix_mysql
  Yet another mysql monitor for zabbix, like percona monitor for zabbix. only test in zabbix 2.2.

## read more:
   http://www.percona.com/doc/percona-monitoring-plugins/1.1/zabbix/index.html

## Note
   Instead of ss_get_mysql_stats.php with mymonitor.pl, the other configuration is similar with ss_get_mysql_stats.php.

    Read more: perldoc mymonitor.pl

    mysql_port.pl: MySQL port discovery for multiport of MySQL, and generate json format strings.

    get_mysql_stats_wrapper.sh: a wrapper for parse MySQL status, runs every 5min.

## Require
    perl-DBI
    perl-DBD-mysql

## Install

Configure MySQL connectivity on Agent

    1. # git clone https://github.com/chenzhe07/zabbix_mysql.git /usr/local/zabbix_mysql 
    
    2. # bash /usr/local/zabbix_mysql/install.sh 192.168.1.2

* note: 192.168.1.2 is your ip address.

Configure Zabbix Server
    
    1. import templates/zabbix_mysql_multiport.xml using Zabbix UI(Configuration -> Templates -> Import), and Create/edit hosts by assigning them “MySQL” group and linking the template “MySQL_zabbix” (Templates tab).

## Note

* The following privileges are needed by monior user. the user and password in get_mysql_stats_wrapper.sh and mymonitor.pl can be changed, but without following privileges:

    PROCESS, SUPER, REPLICATION SLAVE

* As zabbix process running by zabbix user, netstat must run with following command:

    chmod +s /bin/netstat

## Test
```
    # perl  mymonitor.pl --host 10.0.0.10 --port 3300 --items hv
    hv:36968
    # perl  mymonitor.pl --host 10.0.0.10 --port 3300 --items kx
    kx:1070879944

    # php ss_get_mysql_stats.php --host 10.0.0.10 --port 3300 --items hv
    hv:36968
    # php ss_get_mysql_stats.php --host 10.0.0.10 --port 3300 --items kx 
    kx:1070911408

    # zabbix_get -s 10.0.0.10 -p 10050 -k "MySQL.Bytes-received[3300]"
    472339244134
```
