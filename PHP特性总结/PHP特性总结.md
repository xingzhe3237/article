# PHP特性总结

[2018-11-15]()

# [](#PHP特性 "PHP特性")PHP特性

### [](#前言 "前言")前言

打CTF经常会遇到代码审计的题目，一般接触到的都是php型的，做的笔记到处都是，这里就准备把它们全总结到一起。  

## [](#一、数组 "一、数组")一、数组

### [](#0x01-数组的md5 "0x01 数组的md5")0x01 数组的md5

这个大家应该都知道，md5算法对数组加密结果是NULL。  
首先判断username和password是否一致，一致的话提示’Your password can not be your username.’。  
这里就用到了php的一个特性

> php对数组进行md5加密返回的结果都是null

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">&lt;?php</span><br><span class="line">echo md5($_GET['username']);</span><br><span class="line">?&gt;</span><br></pre></td></tr></tbody></table>

运行一下

爆出警告需要一个字符类型的参数，而不是数组

然后我们测试一下上面那个题，输入`username`和`password`都为数组，但是赋值不同

由于两个的赋值不同，所以通过第一个判断，又由于都是数组，php对其进行md5加密后都返回为NULL,所以通过了第二个判断，输出flag。

### [](#0x02-strcmp-函数 "0x02 strcmp()函数")0x02 strcmp\(\)函数

先看这个代码

这里使用strcmp去比较password和flag,如果==0，就给出flag。  
strcmp比较时，如果相等才会返回0，如果不相等返回要么大于0，要么小于0，这里记住一句话：

> strcmp函数只会处理字符串参数，如果给个数组，就会返回NULL，而判断使用的是==，`NULL==0`,这个等式的逻辑值是true。

利用这个漏洞，我们来做这个题

## [](#二、数字的比较 "二、数字的比较")二、数字的比较

### [](#0x01-十六进制与数字 "0x01 十六进制与数字")0x01 十六进制与数字

还是先看题吧

代码的意思就是： 不让输入1到9的数字，但是后面却让比较一串数字，这里想到的就是用进制转换，然后再比较。  
那么将`3735929054`这串数字转换成十六进制是`deadc0de`,然后两个进行比较，比较结果当然是相等的，这样就能成功绕过，得到flag

### [](#0x02-数字运算-一 "0x02 数字运算(一)")0x02 数字运算\(一\)

大致意思就是：POST传入password的值，必须大于12位，必须是非空非TAB，然后password要有大小写数字，字符，陪陪次数要大于6，最后要`$password==42`.

这里直接给出现成的payload:  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">password=42.00e+00000000000</span><br><span class="line">或者</span><br><span class="line">password=420.000000000e-1</span><br></pre></td></tr></tbody></table>

### [](#0x03-数字运算-二 "0x03 数字运算(二)")0x03 数字运算\(二\)

代码中先将变量放到is\_numberic函数中判断，如果是数字或数字字符串则返回true，否则返回false。然后一个判断，如果temp大于1336则显示flag。这里用到了PHP弱类型的一个特性，

> 当一个整形和一个其他类型行比较的时候，会先把其他类型intval再比。

那么输入一个1337a这样的字符串，在is\_numeric中返回true，然后在比较时被转换成数字1337，这样就绕过判断输出flag。

### [](#0x04-MD5的巧合-一 "0x04 MD5的巧合(一)")0x04 MD5的巧合\(一\)

输入password，要求其MD5值为0

> 有一些特定的字符被MD5加密后结果是0e开头的

而URL中0e被当作了科学计数法，0\*10的多少次方都是0  
这样的字符串其实有很多，这里给出几个：

> 240610708  
> QNKCDZO

### [](#0x03-MD5的巧合-二 "0x03 MD5的巧合(二)")0x03 MD5的巧合\(二\)

其中md5运算函数有一个true参数，它的作用是将md5后的hex转换成字符串，这里如果字符串有单引号之类的字符就可以注入了。  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">比如字符串：ffifdyop</span><br><span class="line">md5后，276f722736c95d99e921722cf9ed621c</span><br></pre></td></tr></tbody></table>

  
将其转成字符串的话就是

可以看到起字符串类似于 `'or'6………..`这样的字符串，其中`'or'6`是个永真的条件，如果把它放到查询中就可以where语句的判断，比如我们在url输入`password=ffifdyop`可以看到dump出的数据