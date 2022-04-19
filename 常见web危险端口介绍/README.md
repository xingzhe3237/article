# 常见web危险端口介绍

[2020-03-31]()

## [](#端口及对应的服务和漏洞 "端口及对应的服务和漏洞")端口及对应的服务和漏洞

* 20：FTP服务的数据传输端口
* 21：FTP服务的连接端口，可能存在 弱口令暴力破解

* 22：SSH服务端口，可能存在 弱口令暴力破解
* 23：Telnet端口，可能存在 弱口令暴力破解
* 25：SMTP简单邮件传输协议端口，和 POP3 的110端口对应
* 43：whois服务端口
* 53：DNS服务端口\(TCP/UDP 53\)
* 67/68：DHCP服务端口
* 69：TFTP端口，可能存在弱口令
* 80：HTTP端口，常见web漏洞
* 88：Kerberos协议端口
* 110：POP3邮件服务端口，和SMTP的25端口对应
* 135：RPC服务
* 137/138： NMB服务
* 139：SMB/CIFS服务
* 143：IMAP协议端口
* 161/162: Snmp服务，public弱口令
* 389：LDAP目录访问协议，有可能存在注入、弱口令
* 443：HTTPS端口，心脏滴血等与SSL有关的漏洞
* 445：SMB服务端口，可能存在永恒之蓝漏洞MS17-010
* 512/513/514：Linux Rexec服务端口，可能存在爆破
* 873：Rsync ，可能存在Rsync未授权访问漏洞，传送门：rsync 未授权访问漏洞
* 1080：socket端口，可能存在爆破
* 1099：RMI，可能存在 RMI反序列化漏洞
* 1352：Lotus domino邮件服务端口，可能存在弱口令、信息泄露
* 1433：SQL Server对外提供服务端口
* 1434：用于向请求者返回SQL Server使用了哪个TCP/IP端口
* 1521：oracle数据库端口
* 2049：NFS服务端口，可能存在NFS配置不当
* 2181：ZooKeeper监听端口，可能存在 ZooKeeper未授权访问漏洞
* 2375：Docker端口，可能存在 Docker未授权访问漏洞
* 2601: Zebra ，默认密码zebr
* 3128: squid ，匿名访问（可能内网漫游\)
* 3268：LDAP目录访问协议，有可能存在注入、弱口令
* 3306：MySQL数据库端口，可能存在 弱口令暴力破解
* 3389：Windows远程桌面服务，可能存在 弱口令漏洞 或者 - CVE-2019-0708 远程桌面漏洞复现
* 3690：SVN服务，可能存在SVN泄漏，未授权访问漏洞
* 4440：Rundeck，弱口令admin
* 4560：log4j SocketServer监听的端口，可能存在 log4j\<=1.2.17反序列化漏洞（CVE-2019-17571）
* 4750：BMC，可能存在 BMC服务器自动化RSCD代理远程代码执行\(CVE-2016-1542\)
* 4848：GlassFish控制台端口，可能存在弱口令admin/adminadmin
* 5000：SysBase/DB2数据库端口，可能存在爆破、注入漏洞
* 5432：PostGreSQL数据库的端口
* 5632：PyAnywhere服务端口，可能存在代码执行漏洞
* 5900/5901：VNC监听端口，可能存在 VNC未授权访问漏洞
* 5984：CouchDB端口，可能存在 CouchDB未授权访问漏洞
* 6379：Redis数据库端口，可能存在Redis未授权访问漏洞，传送门：Redis未授权访问漏洞
* 7001/7002：Weblogic，可能存在Weblogic反序列化漏洞，传送门：Weblogic反序列化漏洞
* 7180：Cloudera manager端口
* 8069：Zabbix服务端口，可能存在Zabbix弱口令导致的Getshell漏洞
* 8080：Tomcat、JBoss，可能存在Tomcat管理页面弱口令Getshell，JBoss未授权访问漏洞，传送门：Tomcat管理弱口令页面Getshell
* 8080-8090：可能存在web服务
* 8089：Jetty、Jenkins服务端口，可能存在反序列化，控制台弱口令等漏洞
* 8161：Apache ActiveMQ后台管理系统端口，默认口令密码为：admin:admin ，可能存在CVE-2016-3088漏洞，传送门：Apache ActiveMQ任意文件写入漏洞（CVE-2016-3088）
* 9000：fastcgi端口，可能存在远程命令执行漏洞
* 9001：Supervisord，可能存在Supervisord远程命令执行漏洞\(CVE-2017-11610\)，传送门：Supervisord远程命令执行漏洞\(CVE-2017-11610\)
* 9043/9090：WebSphere，可能存在WebSphere反序列化漏洞
* 9200/9300：Elasticsearch监听端口，可能存在 Elasticsearch未授权访问漏洞
* 10000：Webmin-Web控制面板，可能存在弱口令
* 10001/10002：JmxRemoteLifecycleListener监听的，可能存在Tomcat反序列化漏洞，传送门：Tomcat反序列化漏洞\(CVE-2016-8735\)
* 11211：Memcached监听端口，可能存在 Memcached未授权访问漏洞
* 27017/27018：MongoDB数据库端口，可能存在 MongoDB未授权访问漏洞
* 50000：SAP Management Console服务端口，可能存在 运程命令执行漏洞。
* 50070：Hadoop服务端口，可能存在 Hadoop未授权访问漏洞
* 61616：Apache ActiveMQ服务端口，可能存在 Apache ActiveMQ任意文件写入漏洞（CVE-2016-3088）复现
* 60020：hbase.regionserver.port，HRegionServer的RPC端口
* 60030：hbase.regionserver.info.port，HRegionServer的http端口

* * *

## [](#端口相关的命令（Windows） "端口相关的命令（Windows）")端口相关的命令（Windows）

* netstat -a 显示一个所有的有效连接信息列表，包括已建立的连接（ESTABLISHED ），也包括监听连接请求（LISTENING ）的那些连接，  
  断开连接（CLOSE\_WAIT ）或者处于联机等待状态的（TIME\_WAIT ）等
* netstat -n 以数字形式显示地址和端口号,显示所有已建立的有效连接
* netstat -ano 列出所有端口的情况
* netstat -ano|findstr “80” 查看被占用端口80对应的应用的PID
* tasklist|findstr “80” 查看80端口被哪个进程或程序占用
* 结束该进程或程序：taskkill /f /t /im XX.exe 结束该进程或程序

* * *

## [](#Nmap中常见的服务 "Nmap中常见的服务")Nmap中常见的服务

1.  msmq\?：默认对应的是1801端口，是MSMQ Microsoft Message Queuing（微软消息队列）的简称，是windows系统提供的一个功能，开启了该功能，则默认1801端口打开。该服务暂未发现漏洞。
2.  msrpc：Microsoft Remote Procedure Call（微软远程过程调用）是 Windows 操作系统使用的一个协议。该服务开启时对应端口2103、2105、2107开启。RPC 提供一种内部进程通讯机制，允许在一台电脑上运行的程序无缝的执行远程系统中的代码。
3.  tcpwrapped：端口状态后经常标记tcpwrapped。tcpwrapped表示服务器运行 tcp\_wrappers服务。该服务对应端口10050。tcp\_wrappers是一种应用级防火墙。它可以根据预设，对SSH、Telnet、FTP服务的请求进行拦截，判断是否符合预设要求。如果符合，就会转发给对应的服务进程；否则，会中断连接请求。这说明tcp三次握手已经完成，但是并没有和目标主机建立连接。这表明，虽然目标主机的某项服务是可提供的，但你不在允许访问主机的名单列表中。当大量的端口服务都为tcpwrapped时，这说明可能是有负载均衡或者防火墙阻断了你的连接请求。
4.  Microsoft HTTPAPI httpd 2.0 \(SSDP/UPnP\)：这是SQL Server中的SQL Reporting Service 服务使用的Microsoft HTTPAPI。该服务对应端口5985。
