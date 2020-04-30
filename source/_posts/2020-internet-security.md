---
title: 2020-internet-security by 黑人抬棺队
urlname: 2020-internet-security
tags: 取证
abbrlink: 56887
date: 2020-04-27 21:57:05
---

## 前言

拿到题目挺兴奋的，因为有一点小经验，所以分析起来不算费力，有一些答案很快就找到了。这次对杂项的分析比较快速，毕竟见过类似的，取证的大家收放的速度也是可以的。

## 题目

- [x] 1.	计算机曾经时间是错误的，后经过自动同步调整为了正确的时间。请问该同步的时间为（）（北京时间，格式为yyyyMMddhhmm例如202004250800）
- [x] 2. 计算机系统安装时间为（）（北京时间，格式为yyyyMMddhhmm例如202004250800）
- [ ] 3. 用户xiaohu的开机密码为（） 
- [x] 4. 计算机从曾连接过多个优盘，最早连接的优盘序列号为 （）
- [ ] 5. 请在优盘镜像中找出用户xiaohu壁纸图片（不考虑分辨率是否一致），该图片在优盘中的起始地址为（）（字节）
- [x] 6. 优盘中有个文件拷贝自计算机，拷贝该文件的用户的用户名为（）（字母全部小写）
- [ ] 7. 略
- [ ] 8. 现场取证人员已对一台正处于开机运行状态的Windows系统进行物理内存转储（镜像）并将其压缩为WinMemory.zip，请先解压后再进行分析。该系统可能被勒索病毒感染，通过KDBG扫描检测该系统的Build版本号，版本号为（）(如17763），并使用Volatility Framework 2.6.1版本对疑似进程remsh.exe(PID:9932)进行程序代码转储，导出该程序代码dmp文件，计算其MD5哈希值（字母全部大写）（） ~~~~~~~~~~~~~~注意：两个答案中间用_区分开；
- [ ] 9. usb数据包分析，根据附件提示解出flag，请提交flag{}内的内容；
- [x] 10. 内存取证，根据附件提示解出flag，请提交flag{}内的内容；
- [x] 11. 日志分析，根据附件提示解出flag，请提交flag{}内的内容；
- [x] 12. 数据包分析，根据附件提示解出flag，请提交flag{}内的内容；
- [x] 13. 数据恢复，根据附件提示解出flag，请提交flag{}内的内容；

## 取证1

查看系统日志，找到时间临界处。比如下方的记录编号205。

![](quzheng1_1.png)

找到它的准确时间。`202004192218`

![quzheng3](quzheng1_3.png)

即在202004192218使用Windows Time服务同步时间



## 取证2

由取证一计算系统时间同步后和同步前的差值，查看系统安装时间，系统安装时间+同步前后的差值即实际系统安装时间。

![quzheng](quzheng2_1.png)



## 取证3

没找到，求指教。



## 取证4

查看USB设备使用痕迹，如下：

![quzheng](quzheng4_1.png)

其中USB设备名称：Specific STORAGE DEVICE USB Device为U盘，查看此设备ID如下

![quzheng](quzheng4_2.png)



## 取证6

文件拷贝到U盘的时间是`2020-04-21 10:20 `

![quzheng](quzheng6_1.png)

通过查看系统日志，在此期间用户liu登录系统并使用资源管理器（估计不是liu就是xiaohu）

![quzheng](quzheng6_2.png)

提交liu的时候正确了，那么就是liu了。

## 取证7

略。



## 取证8

硬盘空间不够了，先放弃。



## 取证9

金刚包，求指教。



## 取证10

不知道为什么，一眼看过去直觉告诉我需要`volatility`，所以没有过多的尝试。

![quzheng](quzheng10_1.png)

疑点在桌面一个叫disk.zip的压缩包（禁止套娃），因为是中文系统，逼着我去装了中文适配。

把disk.zip下载下来后打开，disk.img放去还原，得到usb.pcapng，这是一个usb信号记录包，一般键盘信号常驻于第三个字节。

`tshark -r usb.pcapng -T fields -e usb.capdata > usbdata.txt`

使用tshark将数据转文本。借鉴了大佬的[脚本](https://blog.xiafeng2333.top/tools-10/)。

```python
normalKeys = {"04":"a", "05":"b", "06":"c", "07":"d", "08":"e", "09":"f", "0a":"g", "0b":"h", "0c":"i", "0d":"j", "0e":"k", "0f":"l", "10":"m", "11":"n", "12":"o", "13":"p", "14":"q", "15":"r", "16":"s", "17":"t", "18":"u", "19":"v", "1a":"w", "1b":"x", "1c":"y", "1d":"z","1e":"1", "1f":"2", "20":"3", "21":"4", "22":"5", "23":"6","24":"7","25":"8","26":"9","27":"0","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":"<SPACE>","2d":"-","2e":"=","2f":"[","30":"]","31":"\\","32":"<NON>","33":";","34":"'","35":"<GA>","36":",","37":".","38":"/","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}

shiftKeys = {"04":"A", "05":"B", "06":"C", "07":"D", "08":"E", "09":"F", "0a":"G", "0b":"H", "0c":"I", "0d":"J", "0e":"K", "0f":"L", "10":"M", "11":"N", "12":"O", "13":"P", "14":"Q", "15":"R", "16":"S", "17":"T", "18":"U", "19":"V", "1a":"W", "1b":"X", "1c":"Y", "1d":"Z","1e":"!", "1f":"@", "20":"#", "21":"$", "22":"%", "23":"^","24":"&","25":"*","26":"(","27":")","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":"<SPACE>","2d":"_","2e":"+","2f":"{","30":"}","31":"|","32":"<NON>","33":"\"","34":":","35":"<GA>","36":"<","37":">","38":"?","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
output = []
keys = open(r'C:\Users\19308\Desktop\FMP\Case\disk-img\ExportFile\usbdata.txt')
for line in keys:
    try:
        if line[0]!='0' or (line[1]!='0' and line[1]!='2') or line[3]!='0' or line[4]!='0' or line[9]!='0' or line[10]!='0' or line[12]!='0' or line[13]!='0' or line[15]!='0' or line[16]!='0' or line[18]!='0' or line[19]!='0' or line[21]!='0' or line[22]!='0' or line[6:8]=="00":
             continue
        if line[6:8] in normalKeys.keys():
            output += [[normalKeys[line[6:8]]],[shiftKeys[line[6:8]]]][line[1]=='2']
        else:
            # output += ['[unknown]']
            output += ['']
    except:
        pass
keys.close()

flag=0
print("".join(output))
for i in range(len(output)):
    try:
        a=output.index('<DEL>')
        del output[a]
        del output[a-1]
    except:
        pass
for i in range(len(output)):
    try:
        if output[i]=="<CAP>":
            flag+=1
            output.pop(i)
            if flag==2:
                flag=0
        if flag!=0:
            output[i]=output[i].upper()
    except:
        pass
print ('output :' + "".join(output))


```

获得键盘记录

```
'uutdobzipsrn'.hill_decode()aabaaaaabbaabbabcbabbbcccbbbcdcccceccccecccedccdcdcccddcdeddccfcdcdfcdcdccecdcccdcccbcbdbbbbbcbbbbacbaababaaaaabaabaaaaaaaaaaaaaaaaaaaaa
```

希尔解密

`ejcixhwdnegm`

flag应该就是`flag{ejcixhwdnegm}`



## 取证11

access.log拿到后unquote一下获得对人友好的日志。导出

```bash
cat access.log | egrep '!=|<=|>=' > a.txt
```

正则匹配ascii转换一下字符

```python
import re
pattern = re.compile('[!<>]=(\d{2,3})')
with open(r"C:\Users\19308\Desktop\a.txt",'r',encoding='utf-8') as f:
    lst = pattern.findall(f.read())
lst = [chr(int(x)) for x in lst]
print(''.join(lst))
>>>2information_schemasqli2flaguser1flag1flag{w3bvu1lisDanger}
```

flag应该是`flag{w3bvu1lisDanger}`



## 取证12

下载附件，是一个数据包，用wireshark打开一下，过滤下`http`

![quzheng](quzheng12_1.png)

`POST /cmd.php` 看起来非常可疑，一个一个分析太累，导出再看。导出所有html对象，逐个分析，但是看完所有文件的时候，有个可疑的字符串出来了。

![quzheng](quzheng12_2.png)



## 取证13

看见disk0毫不犹豫就拿去还原了。

![quzheng](quzheng13_1.png)

跟进”丢失文件”文件夹，发现这文件

![quzheng](quzheng13_2.png)

里面的内容：

![quzheng](quzheng13_3.png)

打开了p@ss_wd~ 文件

![quzheng](quzheng13_4.png)

根据提示，刚刚拿到的应该是密码。

解压里面的压缩包，获得文件flag.png

![quzheng](quzheng13_5.png)

hex修改一下图高

![quzheng](quzheng13_6.png)



## Web1 

先用御剑扫出后台，然后进入phpadmin数据库后台，在里面找到flag

![quzheng](web1.png)



## 总结

有些东西该记住的还是要记住的，不然忘得很快，这次和叶师傅、付师傅配合得很好，不愧是他们两个，躺着得我非常幸福。还有就是不能过于纠结一些题目，他们往往容易成为时间刺客。

2020年4月27日