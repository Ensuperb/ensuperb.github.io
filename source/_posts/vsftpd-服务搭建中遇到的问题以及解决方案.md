---
title: '[vsftpd]服务搭建中遇到的问题以及解决方案'
urlname: ideatodealproblemof-vsftpd-whenbuild
tags: 杂谈
abbrlink: 20577
date: 2019-10-20 19:00:57
---

## **问题一** **500 OOPS: cannot change directory:/product/ftpfile** 
**500 OOPS: priv_sock_get_cmd** **远程主机关闭连接**

情景：ftp服务打开后,能建立链接，甚至root用户能进入

解决方案：

> 这边的问题纯粹是权限问题，或者是用户的设置问题，首先一开始**弹出的错误是这个** **500 OOPS: cannot change directory:ftpfolder**和文章上的 `/product/ftpfile` 的绝对路径有些出入，所以检查了一下 `/etc/passwd`里面的用户发现是用户的父目录没有设置正确，设置后就能成功用设置的用户登陆进指定的文件夹。

## **问题二** **refusing to run with writable root inside chroot()**

情景： 在我设置了

```
chroot_local_user=YES
chroot_list_enable=NO
```

之后就出现了这个问题

猜测：指定的用户不再指定的白名单里

实际上：这个问题发生在最新的这是由于下面的更新造成的：

> Add stronger checks for the configuration error of running with a writeable root directory inside a chroot(). This may bite people who carelessly turned on chroot_local_user but such is life.

 

解决方案：

> 从2.3.5之后，vsftpd增强了安全检查，如果用户被限定在了其主目录下，则该用户的主目录不能再具有写权限了！如果检查发现还有写权限，就会报该错误。
>
> 要修复这个错误，可以用命令chmod a-w /home/user去除用户主目录的写权限，注意把目录替换成你自己的。或者你可以在vsftpd的配置文件中增加下列两项中的一项：
>
> ```bash
> allow_writeable_chroot=YES
> ```

我的解决方案：直接添加`allow_writeable_chroot=YES`

---

值得参考的文章：

1. [史上最详细的vsftpd配置文件讲解](https://blog.csdn.net/weiyuefei/article/details/51564367)

1. [vsftpd 配置:chroot_local_user与chroot_list_enable详解](https://blog.csdn.net/bluishglc/article/details/42398811)