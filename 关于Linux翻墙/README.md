## Linux翻墙的分析与总结
标题其实还没想好，这篇讲的也不是教大家如何在Linux下翻墙。

Linux下翻墙大家应该都会配置，不会的可以看这篇文章

[https://zone.huoxian.cn/d/844-linuxv2rayn](https://zone.huoxian.cn/d/844-linuxv2rayn)

这里主要讲翻墙后的东西。
连上v2ray之后，浏览器是可以通过代理翻墙了。但是打开一个终端发现并不能通过代理，当然这里也好设置，在终端输入命令：

	export http_proxy=http://127.0.0.1:8889
然后终端就可以走代理了，但是发现ping谷歌还是ping不通，其实这里并不是我们的代理没设置好，也不是终端没走代理，而是因为ping协议是基于ICMP协议，是网络层协议，而我们走的代理无论是http协议还是socks5代理（一个是应用层协议，一个是传输层协议）都是高于网络层的，低层协议不使用高层协议，所以ping协议是不使用http协议或者socks5协议的，所以ping谷歌还是ping不通，但是我们可以通过两种方法来验证我们的终端已经走了代理：

- curl www.google.com
- curl cip.cc

第一种：我们对谷歌使用curl命令，发现是返回了源码的，说明我们是通了外网的。

第二种：`curl cip.cc`命令，会返回你的公网IP地址，也可以证明你的终端是否翻墙了。

最后，如果你非要使用ping命令，可以试试httping
