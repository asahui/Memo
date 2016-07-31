## 1. 汉化组要求的日版与redump版区别
汉化组要求的日版与redump版区别

- 在区间 A6730000-A677FFFF 中  
  - non-redump: 在A6740000-A676FFFF filled with data，这处的data与redump版 在A6740000-A676FFFF 的data是一样的，其余 filled with 00。
  - redump: filled with data


- 结论：A6730000-A673FFFF 与 A6770000-A677FFFF 这两个区间：
  - non-redump: 00
  - redump: filled with data
- 我写了个程序 Jap2Redump，用于这两个 ISO 版本的互相转换。



## 2. 在a9vg下载的non-redump版与redump版区别

a9vg下载的non-redump版 -> redump版 方法
`截去 C6720000-C676FFFF (文件尾) 的 dummy zero block`

Redump版文件大小为 C6720000 (3,329,359,872)



## 3. 汉化补丁所做修改

打汉化补丁
下面这几块(模糊定界是从不同中分出来的，并不是确定的)要分析定位
```
模糊定界            | 文件定界
7000-8881C         | ?                    <- 重构ISO索引
24AB4388-24C64FD7  | 249F0000-24C64FD7    <- ELF 文件更改
24C65000-24CA8390  | 24C65000-24CA8390    <- IOPRP300.IMG 更改
24CE2000-24CF6E26  | 24CE2000-24D2586D    <- FHD文件更改
24E6D000-24EA064C  | 24D26000- file end   <- IMG_BD.BIN 更改
```

尾部添加
C671C000-C79E57FF
由于原来尾部是C671C000-C676FFFF的大小为0x54000 dummy zero block
现在从C671C000写入了新大小为0x12C9800 的 block，估计都有用

更新： 
原来尾部的00不是dummy，是属于IMG_BD.BIN文件的
原来IMG_BD.BIN放在最后的，大小为 0xA19F6000(2711576576) bytes
打补丁后，IMG_BD.BIN大小变为 0xA2CBB800(2731259904) bytes

本来想逆patch汉化补丁，但看起来得有源码且要分析，不然太复杂。



## 4. 汉化组提供的v1.01 4:3与16:9 补丁区别（研究宽屏）

打4：3补丁与16：9补丁后区别

1. I:\PS2ISO\Zero - Shisei no Koe [NTSC-J] [SLPS-25544] - 4_3.iso: 3,349,043,200 字节

2. I:\PS2ISO\Zero - Shisei no Koe [NTSC-J] [SLPS-25544] - 16_9.iso: 3,349,043,200 字节
   Offsets: 十六进制

   829D3:	60	B8
   829D4:	9F	CB
   829D5:	A1	A2
   829D6:	A1	A2
   829D7:	9F	CB
   829D8:	60	B8
   88039:	60	B8
   8803A:	9F	CB
   8803B:	A1	A2
   88040:	EC	77
   88041:	33	59
   24C1106E:80	40  <--- 这里就是关键的宽屏位置 ， ~~上面的几个变动待分析~~

12 不同 被发现。 

上面的变动是修正ISO文件索引的
```python
fp.seek(0x829d2)
fp.write(struct.pack("I" , _LBA_LENGTH * 2048))
fp.write(struct.pack(">I" , _LBA_LENGTH * 2048))
fp.seek(0x88038)
fp.write(struct.pack("I" , _LBA_LENGTH * 2048))
fp.write(struct.pack("I" , 0))
fp.write(struct.pack("I" , _LBA_LENGTH))
```

可以知道代码是修正了两部分，0x829d2与0x88038，其中_LBA_LENGTH * 2048 是 IMG_BD.BIN的大小 _LBA_LENGTH 是 img_file_size / 2048

IMG_BD.BIN大小从 0xA19F6000 变为 0xA2CBB800

也就是说，4：3补丁忘了修正ISO索引，而16:9修正了ISO索引！！！！

真正宽屏代码就只有在
```
0x24C1106C开始
改动
0x3f8000 -> 0x3f4000
0x3f6000 -> 0x3f6000
即Patch里写的
//如果是真机，修改ELF的0022106C
patch=1,EE,0032006C,extended,3F400000
patch=1,EE,00320070,extended,3F600000
```



## 5. 官方4:3还是说校验不正确问题
使用4：3汉化补丁打在要求的原版日版上，还是会提示说校验不通过，但无视打上补丁后，与汉化组提供的汉化镜像对比是一样的。



## 6. 宽屏及其它fix
官方16:9 补丁只修改了一处，使所有东西都变成宽屏。所以正常游戏画面是宽屏了，连动画也宽屏了，但由于动画是本来是4：3，强行宽屏只是拉伸了，所以要fix（模拟器可以直接用 cheat patch，但实机还是得 patch ISO）

```c
//FMV's fix(New)
patch=1,EE,00368140,word,44200000 //44200000
patch=1,EE,00368148,word,3e333333 //3f800000

patch=1,EE,00212808,word,0c09aa90 //c5e00000
patch=1,EE,0021280c,word,c5e10000 //0c09aa90
patch=1,EE,00212810,word,8faf00b0 //e7a00044
patch=1,EE,00212814,word,0200202d //8faf00b0
patch=1,EE,00212818,word,8fae00b4 //0200202d
patch=1,EE,0021281c,word,448f0000 //8fae00b4
patch=1,EE,00212820,word,46800020 //448f0000
patch=1,EE,00212824,word,25effffe //448e0800
patch=1,EE,00212828,word,46010042 //46800020
patch=1,EE,0021282c,word,e7a10044 //25effffe
patch=1,EE,00212830,word,afaf002c //46800860
patch=1,EE,00212834,word,afae0030 //afaf002c
patch=1,EE,00212838,word,3c013f40 //afae0030
patch=1,EE,0021283c,word,44810800 //00000000
patch=1,EE,00212840,word,4601b582 //00000000
patch=1,EE,00212844,word,4600b583 //4600b583
patch=1,EE,00212848,word,448e0800 //00000000
patch=1,EE,0021284c,word,46800860 //00000000
```

```c
// offset: 0x24C59148
// origin
unsigned char data[4] = {
    0x00, 0x00, 0x80, 0x3F
};

// change
unsigned char data[4] = {
	0x33, 0x33, 0x33, 0x3E
};
```

```c
// offset: 0x24B03808
// origin
unsigned char data[72] = {
	0x00, 0x00, 0xE0, 0xC5, 0x90, 0xAA, 0x09, 0x0C, 0x44, 0x00, 0xA0, 0xE7, 0xB0, 0x00, 0xAF, 0x8F, 
	0x2D, 0x20, 0x00, 0x02, 0xB4, 0x00, 0xAE, 0x8F, 0x00, 0x00, 0x8F, 0x44, 0x00, 0x08, 0x8E, 0x44, 
	0x20, 0x00, 0x80, 0x46, 0xFE, 0xFF, 0xEF, 0x25, 0x60, 0x08, 0x80, 0x46, 0x2C, 0x00, 0xAF, 0xAF, 
	0x30, 0x00, 0xAE, 0xAF, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x83, 0xB5, 0x00, 0x46, 
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};

// change
unsigned char data[72] = {
	0x90, 0xAA, 0x09, 0x0C, 0x00, 0x00, 0xE1, 0xC5, 0xB0, 0x00, 0xAF, 0x8F, 0x2D, 0x20, 0x00, 0x02, 
	0xB4, 0x00, 0xAE, 0x8F, 0x00, 0x00, 0x8F, 0x44, 0x20, 0x00, 0x80, 0x46, 0xFE, 0xFF, 0xEF, 0x25, 
	0x42, 0x00, 0x01, 0x46, 0x44, 0x00, 0xA1, 0xE7, 0x2C, 0x00, 0xAF, 0xAF, 0x30, 0x00, 0xAE, 0xAF, 
	0x40, 0x3F, 0x01, 0x3C, 0x00, 0x08, 0x81, 0x44, 0x82, 0xB5, 0x01, 0x46, 0x83, 0xB5, 0x00, 0x46, 
	0x00, 0x08, 0x8E, 0x44, 0x60, 0x08, 0x80, 0x46
};
```



```c
// Focus Effect Off
// offset: 0x24A4706C
// origin
unsigned char data[4] = {
	0xEE, 0x58, 0x05, 0x0C
};

// off
unsigned char data[4] = {
	0x00, 0x00, 0x00, 0x00
};
```

```c
//Bloom offset (fixes bloom glitch)
patch=1,EE,20365008,word,43A30000 // 43A00000 - TC X-offset
patch=1,EE,2036500C,word,43660000 // 43600000 - TC Y-offset

// offset: 0x24C56008
// origin
unsigned char data[8] = {
	0x00, 0x00, 0xA0, 0x43, 0x00, 0x00, 0x60, 0x43
};

// change
unsigned char data[8] = {
	0x00, 0x00, 0xA3, 0x43, 0x00, 0x00, 0x66, 0x43
};
```



**以下patch 默认不开**
```c
//Dither + Ghost post-process Effect Off
//patch=1,EE,00156024,word,00000000 //0c055954
// offset: 0x24A47024
// origin
unsigned char data[4] = {
	0x54, 0x59, 0x05, 0x0C
};
// change
unsigned char data[4] = {
	0x00, 0x00, 0x00, 0x00
};
```

```c
//Disable dark filter (cutscene)
//patch=1,EE,0015609c,word,00000000 //0c05594c
// offset: 0x24A4709C
// origin
unsigned char data[4] = {
	0x4C, 0x59, 0x05, 0x0C
};
// change
unsigned char data[4] = {
	0x00, 0x00, 0x00, 0x00
};
```

```c
//Disable all bloom (speedup, but makes the game seem dull)
//patch=1,EE,00156164,word,00000000 //0c055942
// offset: 0x24A47164
// origin
unsigned char data[4] = {
	0x42, 0x59, 0x05, 0x0C
};
// change
unsigned char data[4] = {
	0x00, 0x00, 0x00, 0x00
};
```

```c
//Disable overbloom (cutscene)
//patch=1,EE,00156100,word,00000000 //0c0558f4
//Decrease overbloom (gameplay)
//patch=1,EE,20364FFC,word,3F400000 //3F800000

// offset: 0x24A47100
// origin
unsigned char data[4] = {
	0xF4, 0x58, 0x05, 0x0C
};
// change
unsigned char data[4] = {
	0x00, 0x00, 0x00, 0x00
};

// offset: 0x24C55FFC
// origin
unsigned char data[4] = {
	0x00, 0x00, 0x80, 0x3F
};
// change
unsigned char data[4] = {
	0x00, 0x00, 0x40, 0x3F
};
```



## 7. 结论

1. 汉化补丁打在汉化组建议的镜像与打在redump版镜像上是一样的，可以说redump镜像还多一点数据，会更加好
2. 汉化组提供的1.01 4:3 汉化补丁忘了修改ISO里面IMG_BD.BIN文件大小
3. 16:9 汉化补丁宽屏就改动一个字节，在24C1106E处从80改到40。这会使用动画也拉伸了，可以手动 Fix，我看有时间我写个 patch 好了。
