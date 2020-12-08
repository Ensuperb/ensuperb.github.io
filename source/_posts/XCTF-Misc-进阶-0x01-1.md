---
title: XCTF-Misc 进阶 0x01+1
urlname: XCTF-Misc-M-0x01
tags: XCTF
abbrlink: 22259
date: 2020-12-08 20:32:33
---

# Misc 进阶 0x01

---

## 神奇的Modbus

wireshark过滤包为modbus，对modbus的数据流进行跟踪。找到flag。

```
PVÀ)#|EYx7@@<À¨À¨öó«##ùýÿpfPå9+(sctf{Easy_Mdbus}
```

提交时对`Modbus`字符串补全。

Refer：[Modbus  zh  wiki](https://zh.m.wikipedia.org/zh-hans/Modbus)



## something_in_image

直接使用Strings.exe工具扫描镜像内的可打印字符串。



## reverseMe

将图片放入word或者PS进行水平垂直翻转各一次。





## wireshark-1

过滤得http报文，跟踪user.php的http流，搜寻关键字`password`。



## pure_color

在线的[stegonline](https://stegonline.georgeom.net/image)有时候确实香，不过真要更加细节的操作需要[Stegsolve](http://www.caesum.com/handbook/stego.htm)。便宜到Grey Bit可以见到清晰的FLAG。





## Aesop_secret

GIF放进photoshop，使所有图层可见，得到可见字符串`ISCC`。再放进十六进制查看器里，观察到文件尾部有疑似Base64的字符串，后了解到是AES密文，放入这个[网站](https://www.sojson.com/encrypt_aes.html)连续解密两次。



---



# Misc 进阶 0x02

---

## a_good_idea

此题有不同解法。

以压缩包的方式打开`a_very_good_idea.jpg`文件，发现有两张图：`to.png` \ `to_do.png`

开始我用盲水印工具无法解出。

> **方案一**
>
> 先导入`to.png`
>
> 再使用stegsolve的`image combiner`导入`to_do.png`进行比较，偏移到`XOR`模式。导入PS把曝光拉到最高（+20）。

> **方案二**
>
> 两张图丢进`beyond Compare`。

> **方案三**
>
> 先导入`to_do.png`
>
> 再使用stegsolve的`image combiner`导入`to.png`进行比较，偏移到`SUB`模式。

> **方案四**
>
> ```python
> from PIL import Image
> img1 = Image.open("to.png")
> img2 = Image.open("to_do.png")
> weight,height = img1.size	# 获取图片长宽
> p1 = img1.load()
> p2 = img2.load()
> 
> new_img = Image.new('RGB',(weight,height))
> p = new_img.load()
> 
> for i in range(weight):
>  	for j in range(height):
>     	# 控制解释：对比两张图像素，不一样的像素输出为白色。
>      	if p1[i, j] != p2[i, j]:
>          	p[i,j] = (255,255,255)
> new_img.show()
> ```

Refer:  [Introduction to Image Analysis -- CTF-Wiki](https://ctf-wiki.github.io/ctf-wiki/misc/picture/introduction-zh/)



## Training-Stegano-1

用十六进制软件打开，结束。





## can_has_stdio

[Brainfuck](https://en.wikipedia.org/wiki/Brainfuck)，前往[在线工具](https://sange.fi/esoteric/brainfuck/impl/interp/i.html)得解。

| 符号                              |                                                              |
| --------------------------------- | ------------------------------------------------------------ |
| >                                 | increment pointer / 指针加一                                 |
| <                                 | decrement pointer / 指针减一                                 |
| +                                 | increment value at pointer / 指针指向的字节的值加一          |
| -                                 | decrement value at pointer / 指针指向的字节的值减一          |
| [                                 | begin loop (continues while value at pointer is non-zero) / 如果指针指向的单元值为零，向后跳转到对应的`]`指令的次一指令处 |
| ]                                 | end loop / 如果指针指向的单元值不为零，向前跳转到对应的`[`指令的次一指令处 |
| ,                                 | read one character from input into value at pointer / 输入内容到指针指向的单元（ASCII码） |
| .                                 | print value at pointer to output as a character / 输出指针指向的单元内容（[ASCII码](https://zh.wikipedia.org/wiki/ASCII码)） |
| #                                 | display debugging info (in debug mode)                       |
| Any other characters are ignored. | ----                                                         |



## János-the-Ripper

没有ZIP伪加密，使用工具`ARCHPR`进行密钥找回。



## Test-flag-please-ignore

以为是md5，转念一想应该是Hex转ASCII。

```python
string = "666c61677b68656c6c6f5f776f726c647d"
step = int(len(string)/2)
b_string = ""
for i in range(0,step):
    tmp = chr(int(string[i*2:i*2+2],16))
    b_string += tmp
print(b_string)
```



## What-is-this

这题看到有两张图第一直觉是盲水印，但是发现这两张图即使是表面上看起来也不是一样的，然后第二直觉是丢进PS里使用混合模式，出flag。

这道题实际上两张图片对应的像素是互补的，则我们可以使用脚本让他们点对点相加试试。

```python
img1 = Image.open("pic1.jpg")
img2 = Image.open("pic2.jpg")
width, height = img1.size

if img1.size == img2.size:
    for x in range(width):
        for y in range(height):
            img1.putpixel((x,y), (img1.getpixel((x, y)) + img2.getpixel((x,y)) ))	# 对像素位的操作
```

