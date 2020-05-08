---
title: 关于SSH的新理解
urlname: new-understand-about-SSH
tags: 杂谈
abbrlink: 19008
date: 2019-07-29 09:31:01
---



## 免密登录

### 理论上说 

> 免密登录需要使用到SSH的非对称性加密(_RSA_)(其实_DES_也可以,但这里主要是针对SSH)中数字签名的特性,即私钥加密公钥解。

> 简单讲：服务器得到公钥，客户端获得私钥。 这样即可完成免密登录。

### 实践操作

在服务器中使用git（因为常用）

`useradd git`

随后密码自拟,

在git的用户目录,通常在`/home/git`

```bash
mkdir .ssh
cd .ssh
ssh-keygen
```

这样即可在服务器中生成私钥与密钥，以及authorized_key。

| 文件名          | 啥             |
| --------------- | -------------- |
| id_rsa.pub      | 公钥           |
| id_rsa          | 私钥           |
| authorized_keys | 已认证的公钥？ |

ssh-keygen有多种参数可供选择,这里不赘述。[ 想获得更多？](https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

接下来 `cat id_rsa.pub > authorized_keys`

通过各种手段（例如windows 的 winscp之类的）将id_rsa 下载下来 放在 `C:\user\ *** \.ssh\`

*顺便看看这个文件夹里面有没有一个叫 known_hosts 的文件* ，如果有就没事，如果没有，会不会是第一次连接的时候否定了什么呢？

> ssh会把你每个你访问过计算机的公钥(public key)都记录在~/.ssh/known_hosts。当下次访问相同计算机时，OpenSSH会核对公钥。如果公钥不同，OpenSSH会发出警告， 避免你受到DNS Hijack之类的攻击。 [From:liuyanerfly](https://www.cnblogs.com/liuyanerfly/p/9668426.html)

接下来就是一顿测试操作: 上powershell或cmd（只要有ssh就行）

`ssh -v git@yourIP:SSHport`

如果不用提示输入密码，成功。当然，很多人都会在这里有疑惑：我怎么还是需要输入密码？这就要牵扯到权限。

。这是一个非常值得提的通过大量实践证明的问题解决方案。

```bash
chmod 700 .ssh			 #用户全权，其他人没有任何权限
chmod 644 authorized_keys #除用户读写外，组与其他人均为只读
```

在服务端执行了以上指令后，再来客户端执行看看？

![](ssh-after.png)

![](ssh-before.png)

![](authorized_keys.png)





## 权限 

如果是使用git 作为免密登录时通常会需要sudo权限。

在root 的情况下 

```bash
chmod 740 /etc/sudoers 
vim /etc/sudoers
...
```

在 `# User privilege specification` 下面增添一条

```
git    ALL=(ALL:ALL) ALL
```

(通常熟悉vim的同学可以`yy`以及`p`快速一顿复制黏贴,这个真的强)

记得wq!退出,之后改回权限 `chmod 440 /etc/sudoers`

------

### Git Hook?

啥? Hexo d 部署没反应? 

- 会不会时hooks 里面的 post-receive 写错了?

- emmmm,没有错的话,会不会是 post-receive 没有执行权限呢?

  赶紧 `chmod +x post-receive ` 一波。


