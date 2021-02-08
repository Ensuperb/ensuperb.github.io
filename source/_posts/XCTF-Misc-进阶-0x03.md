---
title: XCTF-Misc 进阶 0x03
urlname: XCTF-Misc-M-0x03
tags: XCTF
abbrlink: 14830
date: 2020-12-13 15:18:55
---

# Misc 进阶 0x03

---

## hit-the-core

> 知识点：字符串隐写、编程能力

使用Strings指令扫描此文件可打印字符。

使用hexdump -C 能看到更详细的内容。

发现可疑字符串，留意大写（后来发现是随着前面大写的顺序顺下去)

从第四个字符开始，每隔5个字符取一个字符。

```
cvqAeqacLtqazEigwiXobxrCrtuiTzahfFreqc{bnjrKwgk83kgd43j85ePgb_e_rwqr7fvbmHjklo3tews_hmkogooyf0vbnk0ii87Drfgh_n kiwutfb0ghk9ro987k5tfb_hjiouo087ptfcv}
```



## 快乐游戏题

> 知识点：EXE逆向？？？

完了几把，赢了。

但是这真的是正确的打开方式？先放放。



## banmabanma

> 知识点：条形码

看到此图，联想到**条形码**，使用手机`二维码阅读器`扫描得到结果。

编码格式：`CODE_39`

编码信息：`FLAG IS TENSHINE`

当然也有[在线识别编码的工具](https://online-barcode-reader.inliteresearch.com/)。



## embarrass

> 知识点：字符串、流量分析

Wireshark打开文件，分析半天，没什么收获。

用`010editor`打开，搜索flag得到答案。



## Ditf

> 知识点：图片隐写、PNG格式、流量分析

拿到图片，使用010editor打开发现下方有rar头，使用foremost分离，压缩包有密码，起初认为是伪加密，修改后发现不是，查看原图片有没有隐藏信息。使用`stegsolve`没发现什么线索，使用`TweakPNG`检查图片的CRC。报如下错误：

```
Incorrect crc for IHDR chunk (is 38165a34, should be 7ebcdb1a)
Invalid chunk type found at fhle position 989714.This may indicate garbage at the end of the file.
```

尝试使用脚本爆破一下图片长宽：

```python
import zlib
import binascii
import struct
import os
os.chdir(r"xxx")
#读文件
file = "00000000.png"
fr = open(file,'rb').read()
data = bytearray(fr[12:29])
crc32key = eval('0x'+str(binascii.b2a_hex(fr[0x1d:0x21]))[2:-1])
# crc32key = eval(str(fr[29:33]).replace('\\x','').replace("b\'",'0x').replace("\'",''))
#crc32key = 0xCBD6DF8A #补上0x，copy hex value
#data = bytearray(b'\x49\x48\x44\x52\x00\x00\x01\xF4\x00\x00\x01\xF1\x08\x06\x00\x00\x00')  #hex下copy grep hex 
n = 4095 #理论上0xffffffff,但考虑到屏幕实际，0x0fff就差不多了
for w in range(n):#高和宽一起爆破
    width = bytearray(struct.pack('>i', w))#q为8字节，i为4字节，h为2字节
    for h in range(n):
        height = bytearray(struct.pack('>i', h))
        for x in range(4):
            data[x+4] = width[x]
            data[x+8] = height[x]
            #print(data)
        crc32result = zlib.crc32(data)
        if crc32result == crc32key:
            print(width,height)
            #写文件
            newpic = bytearray(fr)
            for x in range(4):
                newpic[x+16] = width[x]
                newpic[x+20] = height[x]
            fw = open(file+'.png','wb')#保存副本
            fw.write(newpic)
            fw.close()
```

得出正确的PNG高度，下方隐藏了RAR的密码。

使用wireshark打开解压的文件，过滤取HTTP报文。发现一个可疑的URL，名为`/kiss.png`，下方有疑似Base64密文，解开即可。



## Hear-with-your-Eyes

> 知识点：频谱图

使用Adobe家的`Audition`打开，查看频谱可以得到看到答案。

[![re0Izj.png](https://s3.ax1x.com/2020/12/13/re0Izj.png)](https://imgchr.com/i/re0Izj)

