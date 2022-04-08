# PHP反序列化

[2020-03-23]()

## [](#前言 "前言")前言

考研用了一年，好久没写博客了，安全也很久没搞了，最近重新学了一遍以前自己的笔记，今天学到PHP序列化和反序列化，之前对这个概念就模模糊糊的，今天正好弄清楚。

### [](#0x00：PHP序列化 "0x00：PHP序列化")0x00：PHP序列化

#### [](#序列化函数：serialize "序列化函数：serialize()")序列化函数：serialize\(\)

所有php里面的值都可以使用函数serialize\(\)来返回一个包含字节流的字符串来表示。序列化一个对象将会保存对象的所有变量，但是不会保存对象的方法，只会保存类的名字。

**变量所储存的数据是内存数据，在程序执行结束时，内存数据便会立即销毁；而文件、数据库是“持久存储”，因此PHP序列化就是将内存的数据保存到文件中的过程。**

1.  \$s = serialize\(\$变量\); //该函数将变量数据进行序列化转换为字符串
2.  file\_put\_contents\(‘./目标文本文件’, \$s\); //将\$s保存到指定文件

#### [](#For-example "For example")For example

```
<?php
class User
{
    // 创建类数据

    public $age = 0;
    public $name = '';

    // 输出数据

    public function PrintData()
    {
        echo 'User ' . $this->name . ' is ' .$this->age . ' years old.';
        echo "\n";
    }
}

// 创建一个对象

$usr = new User();

// 设置数据

$usr->age = 20;
$usr->name = 'lemon';

// 输出数据

$usr->PrintData();

// 输出序列化之后的数据

echo serialize($usr);
echo "\n";
?>
```

输出序列化后的结果

1.  User lemon is 20 years old.
2.  O:4:”User”:2:\{s:3:”age”;i:20;s:4:”name”;s:5:”lemon”;\}

可以看到序列化一个对象后会保存对象的所有变量，并且发现序列化后的结果都有一个字符，这些字符都是一下字母的缩写。

* a array
* b boolean
* d double
* i integer
* o common object
* r reference
* s string
* C custom object
* O class
* N null
* R pointer reference
* U unicode string

了解了缩写字母的含义，就可以解读序列化后的含义

1.  O:4:”User”:2:\{s:3:”age”;i:20;s:4:”name”;s:5:”lemon”;\}
2.  对象类型:长度:”类名”:类中变量的个数:\{类型:长度:”值”;类型:长度:”值”;……\}

通过这个例子，应该能理解序列化函数serialize\(\)的功能

### [](#0x01：PHP反序列化 "0x01：PHP反序列化")0x01：PHP反序列化

函数：unserialize\(\)

> unserialize\(\)对单一的已序列化的变量进行操作，将其转回PHP的值。在解序列化一个对象前，这个对象的类必须在解序列化之前定义。

简单理解起来就是**将序列化过存储到文件中的数据，恢复到程序代码的变量表示形式的过程**，恢复到变量序列化之前的结果。

```
$s = file_get_contents(‘./目标文本文件’); //取得文本文件的内容（之前序列化过的字符串）
$变量 = unserialize($s); //将该文本内容，反序列化到指定的变量中
```

#### [](#For-example-1 "For example")For example

```
<?php
    // 某类
    class User
    {
        // Class data
        public $age = 0;
        public $name = '';

        // Print data

        public function PrintData()
        {
            echo 'User ' . $this->name . ' is ' . $this->age . ' years old.';
            echo "\n";
        }
    }
    // 重建对象
    $usr = unserialize('O:4:"User":2:{s:3:"age";i:20;s:4:"name";s:5:"lemon";}');

    // 调出PrintData输出数据
    $usr->PrintData();
?>
```

输出结果：

> User lemon is 20 years old.

**注意：在解序列化一个对象前，这个对象的类必须在解序列化之前定义。否则会报错**

在先知上看大师傅举得例子对序列化和反序列化的介绍，也很好理解。

```
<?php
class A{
    var $test = "demo";
}

$a = new A(); //生成a对象
$b = serialize($a); //序列化a对象为b
$c = unserialize($b); //反序列化b对象为c

print_r($b); //输出序列化之后的值
echo "\n";
print_r($c->test); //输出对象c中test的值：demo

?>
```

### [](#0x02：PHP反序列化漏洞 "0x02：PHP反序列化漏洞")0x02：PHP反序列化漏洞

在学习漏洞前，先来了解一下PHP魔法函数，对接下来的学习会很有帮助

> PHP将所有以\_\_\(两个下划线\)开头的类方法保留为魔术方法

1.  \_\_construct 当一个对象创建时被调用，
2.  \_\_destruct 当一个对象销毁时被调用，
3.  \_\_toString 当一个对象被当作一个字符串被调用。
4.  \_\_wakeup\(\) 使用unserialize时触发
5.  \_\_sleep\(\) 使用serialize时触发
6.  \_\_destruct\(\) 对象被销毁时触发
7.  \_\_call\(\) 在对象上下文中调用不可访问的方法时触发
8.  \_\_callStatic\(\) 在静态上下文中调用不可访问的方法时触发
9.  \_\_get\(\) 用于从不可访问的属性读取数据
10.  \_\_set\(\) 用于将数据写入不可访问的属性
11.  \_\_isset\(\) 在不可访问的属性上调用isset\(\)或empty\(\)触发
12.  \_\_unset\(\) 在不可访问的属性上使用unset\(\)时触发
13.  \_\_toString\(\) 把类当作字符串使用时触发,返回值需要为字符串
14.  \_\_invoke\(\) 当脚本尝试将对象调用为函数时触发

这里只列出了一部分的魔法函数，具体可见  
<https://www.php.net/manual/zh/language.oop5.magic.php>

下面通过一个例子来简单了解一下魔法函数被自动调用的过程

```
<?php
class test{
 public $varr1="abc";
 public $varr2="123";
 public function echoP(){
  echo $this->varr1."<br>";
 }
 public function __construct(){
  echo "__construct<br>";
 }
 public function __destruct(){
  echo "__destruct<br>";
 }
 public function __toString(){
  return "__toString<br>";
 }
 public function __sleep(){
  echo "__sleep<br>";
  return array('varr1','varr2');
 }
 public function __wakeup(){
  echo "__wakeup<br>";
 }
}

$obj = new test();  //实例化对象，调用__construct()方法，输出__construct
$obj->echoP();   //调用echoP()方法，输出"abc"
echo $obj;//obj对象被当做字符串输出，调用__toString()方法，输出__toString
$s =serialize($obj);  //obj对象被序列化，调用__sleep()方法，输出__sleep
echo unserialize($s);  //$s首先会被反序列化，会调用__wake()方法，被反序列化出来的对象又被当做字符串，就会调用_toString()方法。
// 脚本结束又会调用__destruct()方法，输出__destruct
?>
```