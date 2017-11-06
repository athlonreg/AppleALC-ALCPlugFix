---
title:      AppleALC-ALCPlugFix
date:       2017-11-06
categories: Hacintosh
description: 简单制作 AppleALC 驱动声卡并解决耳机、外放切换、麦克风无输入以及耳机杂音、唤醒无声等问题
tags:
    - Hacintosh
    - 黑苹果
---

## 前言
> 在`vit9696`推出`AppleALC`后，经过越来越多人仿冒声卡得到的数据的提交整理，`AppleALC`的数据越来越集中，也使声卡驱动变得越来越简单。
> 
> 本文提供使用`AppleALC`驱动声卡的思路以及常见错误和驱动后的问题进行修复。

## 知识小普及
> 首先要知道声卡仿冒的原理
> 

## 前期准备
- 需要下载`Xcode`
> 在`AppleStore`下载

- 下载`AppleALC`
> 通过终端执行下面的命令，会下载到你的用户根目录

```
$ git clone https://github.com/vit9696/AppleALC
```

- 下载`Lilu`
> 在下面的网址下载`DEBUG`和`RELEASE`

```
https://github.com/vit9696/Lilu/releases
```

- 提取`Codec`
> 这一步需要用到`Linux`环境，首先下载`Ubuntu`镜像，我个人提供一个

```
链接:http://pan.baidu.com/s/1i47I0jN  密码:de4v
```

> 当然也可以去官网下载，下载好镜像之后，将`U`盘格式化为`Fat32`，然后将镜像解压到`U`盘，重启以`U`盘启动，选择试用进入`Ubuntu`系统界面，打开终端输入

```
$ cat /proc/asound/card0/codec#0 > ~/Desktop/Codec.txt
```

> 执行完后会将`Codec.txt`提取到桌面，将其保存到`U`盘

**PS: 一般来说提取的`Codec`会有`10k`左右大小，如果大小不够，很有可能提取错误，可以尝试以下命令**

```
$ cat /proc/asound/card0/codec#1 > ~/Desktop/Codec.txt
```

> 或者

```
$ cat /proc/asound/card0/codec#2 > ~/Desktop/Codec.txt
```

> 或者

```
$ cat /proc/asound/card1/codec#0 > ~/Desktop/Codec.txt
```

## `Codec.txt`的数据整理
- 记录`Address`和`Vendor Id`值

> `Codec.txt`开头部分

![2017-11-06-05](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-05.png)


- 提取节点信息

> 搜索`Pin Default`记录`Node`和节点描述，以我的为例

![2017-11-06-03](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-03.png)

**PS: N/A节点无效无需整理，如下面的不需要整理**

![2017-11-06-04](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-04.png)

- 记录`HP Out at Ext Left`节点的`tag`和`enable`值

> PS: Unsolicited Response的设定是为了解决耳机和外放切换的问题。
需设定 HP Out at Ext 所在的节点，并且节点信息中有Unsolicited: tag=xx, enabled=1 这一行。
设定命令为Address+NodeID+708+<设定值>
> 
> 设定值是8位的一个二进制数，记作a7 a6 a5 a4 a3 a2 a1 a0，推算出此二进制数之后，四四拆分转换为2位的十进制数加1即为此设定值
> 
> a7表示enabled。
> a6=0，没具体应用，不用管。
> a5~a0，存放tag。**

- 记录`Mic at Ext`节点的`Pin-ctls`值

> PS: Pin Control Widget的设定是为了解决耳机杂音的。
> 
> 需要设定的是 Mic at Ext  所在的节点，设定的值可以从codec dump中读取。
> 
> 设定命令是 Address+NodeID+707+Pin-ctls值。

> 整理完后，会得到如下所示的数据集合

![2017-11-06-06](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-06.png)

**PS: 常见的节点设备描述如下：**
> `Mic at Ext`--线路输入(耳麦)
> 
> `Mic at In`--内建输入
> 
> `HP Out`--耳机扬声器
> 
> `Speaker`--内建扬声器(外放)


<p align="center"><b>至此，`Codec`数据的整理到此结束</b></p>

## `AppleALC`中数据的筛选
> 依次打开下载的`AppleALC`源码里面`/AppleALC/Resources/PinConfigs.kext/Contents`下的`Info.plist`
> 
> 搜索之前记录的`CodecID`

![2017-11-06-07](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-07.png)

> 记录搜索到的所有的`ID`中的`configdata`，如下整理

![2017-11-06-08](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-08.png)

> 每一行的每一串的第一个字符代表`Address`值

> 每一行的第三串倒数第二个字符代表这个节点的设备描述，具体对应关系如下

![2017-11-06-01](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-01.png)

> 根据上图设备对应关系和`Address`值排除掉无效的`ID`数据

## 开工
- 将下载的`Debug`的`Lilu.kext`放进 AppleALC 源码根目录；
- 删除`/AppleALC/Resources`中多余文件夹，只留下你的声卡型号文件夹、`Pinconfigs.kext`以及四个`plist`文件，以`cx20751`这个声卡为例剩下如下文件

![ALC-2](http://ovefvi4g3.bkt.clouddn.com/ALC-2.png)

- 然后打开计算器，显示为编程器

![ALC-3](http://ovefvi4g3.bkt.clouddn.com/ALC-3.png)

- 打开你整理的`Codec`，找到`Vendor Id`，拷贝后面的字符串，在计算器选中十六进制，粘贴这个字符串

![ALC-4](http://ovefvi4g3.bkt.clouddn.com/ALC-4.png)

- 然后选中十进制，就换转换成十进制形式

![ALC-5](http://ovefvi4g3.bkt.clouddn.com/ALC-5.png)

- 拷贝这个十进制数，打开`/AppleALC/Resources/CX20751_2/Info.plist`(此处的`CX20751_2`需要换成你的声卡型号)，把`codecid`换成刚才拷贝的十进制数

![ALC-6](http://ovefvi4g3.bkt.clouddn.com/ALC-6.png)

- 保存退出，右键`PinConfigs.kext`显示包内容，打开里面的`Info.plist`，搜索刚才拷贝的十进制数，记下`LayoutID`数据，有几个记几个，都记下来，然后将`IOKitPersonalities->HDA Hardware Config Resource->HDAConfigDefault`中的其他型号删除，(为避免出错，这里的删除可以不操作，删除只是为了精简做出来的`AppleALC`)，保存退出。

- 然后双击打开 AppleALC 中的工程文件：

![ALC-7](http://ovefvi4g3.bkt.clouddn.com/ALC-7.png)

- 按图示操作

![ALC-8](http://ovefvi4g3.bkt.clouddn.com/ALC-8.png)

- 点击右面的`export` 

![ALC-9](http://ovefvi4g3.bkt.clouddn.com/ALC-9.png)

- next 

![ALC-10](http://ovefvi4g3.bkt.clouddn.com/ALC-10.png)

- `where`填上桌面，点`export`就生成`AppleALC`在桌面上了，一层一层打开它，将其中的`AppleALC.kext`放到`clover`驱动目录，注意之前下载的`Release`的`Lilu`也要放到`clover`驱动目录，最后不要忘了在`config`注入`LayoutID`：

![ALC-11](http://ovefvi4g3.bkt.clouddn.com/ALC-11.png)

- 如图`Audio`处写上刚才记下的`LayoutID`，如果重启后你的声卡不能驱动，或者已驱动但是没有输入或输出，这时很有可能是该`LayoutID`对应的`configdata`数据有错误，这时就挨个尝试刚才记录的所有`ID`。

<p align="center"><b>至此， AppleALC 驱动声卡部分结束。</b></p>

## 关于解决耳机与内建输入的切换
> 如果驱动工作完成后，声卡可以工作，但不能自动切换，请接着往下看

-  打开终端，输入 

```
git clone https://github.com/goodwin/ALCPlugFix 
```
  
- 回车就将`ALCPlugFix`下载到了你的用户目录，打开此目录中的`ALCPlugFix`中的`main.m`下拉到最下方，注意这一部分：

![ALC-12](http://ovefvi4g3.bkt.clouddn.com/ALC-12.png)

- 下载[`had-tools`](https://github.com/athlonreg/ALCPlugFix),将`codec`复制到`had-tools`目录，打开终端，`cd`到此目录，输入 

```
./widget_dump.sh 
```

- 回车

![ALC-13](http://ovefvi4g3.bkt.clouddn.com/ALC-13.png)

- 找到`nid = 0x19`和`nid = 0x1a`，这里我的`19`为 `line in`，`1a`为`mic in`，记录下最后两位，我的是`04`和`24`就这么改

**PS: 这里需要设定的节点数据分别是`line in`和`mic in`，其他节点无效，思想就是捕获系统在不插耳机时由于其输入输出正常的这两个节点的输出值，然后就可以知道插耳机正常应该输出的值，进而利用`AppleALC`守护进程动态守护。如果出现插耳机正常，不插无效的情况，请按照这个思路反过来操作，思想都是一样的。其中图上的高亮处最上面一部分是默认情况，中间是耳机移除，最下面是耳机插入，请结合自身使用情况合理设定**

![ALC-14](http://ovefvi4g3.bkt.clouddn.com/ALC-14.png)

- 保存退出，双击按照`AppleALC`的编译方法编译这个

![ALC-15](http://ovefvi4g3.bkt.clouddn.com/ALC-15.png)

- 然后将生成的`ALCPlugFix`替换`alc_fix`中的`ALCPlugFix`，终端`cd`到`alc_fix`目录，执行 

```
./install.sh
```
    
- 耳机就可以自动切换了，三节点的朋友运气好的话杂音应该也解决了，这时插入耳机在执行 

```
./widget_dump.sh 
```
- 就可以发现之前的`19`和`1a`后面的数据反过来了

> 其实这里的数据就是侦测耳机插拔状态，向系统发送相关指令来做到切换正常
> 
> 需要设定的就是两个节点`Mic at Ext`(有时是`Line In`)和`Mic at In`
> 
> `Mic at Ext`(有时是`Line In`)表示线路输入，即`耳麦`，多为`0x19`节点
> 
> `Mic at In`为内建输入

- **PS: 如果`widget_dump.sh`脚本得不到想要的结果，请在[点击这里](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/)下载`CodecCommander`，将`Release`里面的`hda-verb`利用命令拷贝到`/usr/bin/`**

![ALC-16](http://ovefvi4g3.bkt.clouddn.com/ALC-16.png)

- 如果不行重启一次应该就好了。

<p align="center"><b>至此，解决耳机与内建输入的切换部分结束。</b></p>

## 关于唤醒无声
> 可以先尝试下面两个驱动
> 
> [CodecCommander](https://github.com/RehabMan/EAPD-Codec-Commander)
> [EAPDFix](http://pan.baidu.com/s/1nuTUVS9) 提取密码:`w4yr`

**PS: 关于`CodecCommander`，可以[点击这里](https://github.com/RehabMan/EAPD-Codec-Commander/blob/master/README.md)参考`RehabMan`的说明**

> 另外四叶草助手中也提供了相关选项(`ResetHDA`和`dartweak`)，但是本人没有亲自尝试，大家可以自行尝试这些组合：

![2017-11-06-09](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-09.png)

![2017-11-06-10](http://ovefvi4g3.bkt.clouddn.com/2017-11-06-10.png)

<p align="center"><b>本人对唤醒无声研究不多，至此，解决耳机与内建输入的切换部分结束，欢迎补充。</b></p>

## 关于万能声卡
> 很多人不喜欢万能声卡，说万能有杂音(但是`有一些人的AppleALC`也可能有杂音)，甚至说用万能声卡是不完美的表现，其实我认为不然，万能是一种很好的渠道，有不少人用万能声卡，他们的切换、输入甚至`HDMI`都能很好的输出，而且杂音也是可以减小的。
> 
> 这里我给出官方的项目地址，有兴趣的可以试一下
> 
> https://sourceforge.net/projects/voodoohda/

**PS: 需要注意的是有人用的是`VoodooHDA.pkg`安装的，这种情况需要将`AppleHDADisabler.kext`放到`Clover`的驱动目录来禁用`AppleHDA`，否则很有可能会造成`KP`五果，所以我建议直接用`kext`，让`Clover`注入器注入即可。**

## Credit
[vit9696](https://github.com/vit9696)
[goodwin](https://github.com/goodwin/ALCPlugFix)
[RehabMan](https://github.com/RehabMan)
[Dolnor](https://github.com/Dolnor/EAPD-Codec-Commander)
[daliansky](https://github.com/daliansky)
[VoodooHDA](https://sourceforge.net/projects/voodoohda/)

## 持续更新
