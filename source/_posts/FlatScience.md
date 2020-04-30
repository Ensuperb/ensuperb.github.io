---
layout: FlatScience From XCTF
urlname: FlatScience-From-XCTF
title: FlatScience
date: 2020-04-24 22:23:58
tags: 安全
top: true
---

# [Web] FlatScience

## 思路

拿到网址，把连接都点了一遍，有一些PDF文件，其他还没发现有啥。

康康robots.txt 。有以下发现。

```
User-agent: *
Disallow: /login.php
Disallow: /admin.php
```

在/login.php中发现存在注入。 /admin.php比较顽固。

在login里尝试里一段时间的注入后，有点不理想，点开html源码看看，发现如下

`  <!-- TODO: Remove ?debug-Parameter! --> ` 加个参数在login后面看看，源码给露出来了。

```php
<?php
if(isset($_POST['usr']) && isset($_POST['pw'])){
        $user = $_POST['usr'];
        $pass = $_POST['pw'];

        $db = new SQLite3('../fancy.db');
        $res = $db->query("SELECT id,name from Users where name='".$user."' and password='".sha1($pass."Salz!")."'"); 
    	// 上面就是注入点
    if($res){
        $row = $res->fetchArray();
        // 成功就返回得到的数组。返回到row
    }
    else{
        echo "<br>Some Error occourred!";
    }
    if(isset($row['id'])){
        	// 如果返回了row， 那么就在cookie输出。
            setcookie('name',' '.$row['name'], time() + 60, '/');
            header("Location: /");
            die();
    }
}
if(isset($_GET['debug']))
highlight_file('login.php');
?> 
```



得到Cookies

```json
Set-Cookie: name=+CREATE+TABLE+Users%28id+int+primary+key%2Cname+varchar%28255%29%2Cpassword+varchar%28255%29%2Chint+varchar%28255%29%29; expires=Tue, 21-Apr-2020 10:25:35 GMT; Max-Age=60; path=/
```

URL转码清理一下可以看到这个表的结构：

```sqlite
CREATE TABLE Users(
    id int primary key,
    name varchar(255),
    password varchar(255),
    hint varchar(255)
);
```

| id   | name   | password                                 | hint                          |
| ---- | ------ | ---------------------------------------- | ----------------------------- |
| 1    | admin  | 3fab54a50e770d830c0416df817567662a9dc85c | my+fav+word+in+my+fav+paper?! |
| 2    | fritze | 54eae8935c90f467427f05e4ece82cf569f89507 | my+love+is aci? (wh_9651002)  |
| 3    | hansi  | 34b0bb7c304949f9ff2fc101eef0f048be10d3bd | password                      |

按照hint里面的说法，这个密码是他最喜欢的词，而且还是在论文里边？！

好，懂了。把全站的PDF文件整下来，然后一个一个词碰撞试试。

### 程序思路

1. 递归获取站点的所有超链接资源，放入集合。（为什么是集合，集合它不重复呀）
2. 得到的集合用正则把PDF资源筛选出来，逐个下载到本地。
3. 把PDF转成文本后遍历文本里的所有词，加盐后使用SHA-1加密，拿加密后的值与得到的HASH值进行碰撞。

碰撞结果 `ThinJerboa`

![Result](Result.jpg)

（爬虫的写得太破就不放出来了。）

## 总结

递归的结束条件一定要掌控好，不然既浪费时间也容易造成内存问题。数据库的查询语句要扎实才容易分析出破绽所在。最后，我这里是碰巧就在Robots.txt撞到了两登录入口，普通情况下还是扫描一下为常规操作。

## 所使用的Python库

- 重点
  - pdfplumber
  - bs.beautifulsoup
  - re
  - hashlib
  - requests
- 次要
  - os
  - time

## 知识点

#数据库 #SQLite #递归 #PDF转纯文本 #碰撞



