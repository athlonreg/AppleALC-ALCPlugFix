# AppleALC-ALCPlugFix
简单制作AppleALC驱动声卡并解决耳机、外放切换以及麦克风无输入
首先下载AppleALC源码，打开终端输入： git clone https://github.com/vit9696/AppleALC 回车之后就将AppleALC下载到了你的用户根目录

![ALC-1](http://ovefvi4g3.bkt.clouddn.com/ALC-1-1.png)

然后下载Lilu：链接:http://pan.baidu.com/s/1c2yB3vq 密码:if62

将下载的Lilu中Debug里面的Lilu.kext放进AppleALC源码根目录，然后删除/AppleALC/Resources中多余文件夹，只留下你的声卡型号文件夹、Pinconfigs.kext以及四个plist文件，以cx20751为例剩下如下文件

![ALC-2](http://ovefvi4g3.bkt.clouddn.com/ALC-2-1.png)

然后打开计算器，显示为编程器

![ALC-3](http://ovefvi4g3.bkt.clouddn.com/ALC-3-1.png)

打开你的Codec，找到Vendor Id，拷贝后面的字符串，在计算器选中十六进制，粘贴这个字符串

![ALC-4](http://ovefvi4g3.bkt.clouddn.com/ALC-4-1.png)

然后选中十进制，就换转换成十进制形式

![ALC-5](http://ovefvi4g3.bkt.clouddn.com/ALC-5-1.png)

拷贝这个十进制数，打开/AppleALC/Resources/CX20751_2/Info.plist（此处的CX20751_2需要换成你的声卡型号），把codecid换成刚才拷贝的十进制数

![ALC-6](http://ovefvi4g3.bkt.clouddn.com/ALC-6-1.png)

保存退出，右键PinConfigs.kext显示包内容，打开里面的Info.plist，搜索刚才拷贝的十进制数，记下LayoutID数据，有几个记几个，都记下来，然后将IOKitPersonalities->HDA Hardware Config Resource->HDAConfigDefault中的其他型号删除，（为避免出错，这里的删除可以不操作，删除只是为了精简做出来的AppleALC），保存退出。

然后双击打开AppleALC中的工程文件：

![ALC-7](http://ovefvi4g3.bkt.clouddn.com/ALC-7-1.png)

按图示操作

![ALC-8](http://ovefvi4g3.bkt.clouddn.com/ALC-8-1.png)

点击右面的export

![ALC-9](http://ovefvi4g3.bkt.clouddn.com/ALC-9-1.png)

next

![ALC-10](http://ovefvi4g3.bkt.clouddn.com/ALC-10-1.png)

where填上桌面，点export就生成AppleALC在桌面上了，一层一层打开它，将其中的AppleALC.kext放到clover驱动目录，注意之前下载的Lilu里面的Release中Lilu.kext也要放到clover驱动目录，最后不要忘了在config注入LayoutID：

![ALC-11](http://ovefvi4g3.bkt.clouddn.com/ALC-11-1.png)

如图Audio处写上刚才记下的LayoutID 到这里AppleALC的制作就完了，如果重启后你的声卡不能驱动，就挨个试刚才记录的所有ID，如果能驱动但无法做到插入耳机自动切换，接着往下看： 打开终端，输入 git clone https://github.com/goodwin/ALCPlugFix 回车就将ALCPlugFix下载到了你的用户目录，打开此目录中的 ALCPlugFix 中的 main.m 下拉到最下方，注意这一部分：

![ALC-12](http://ovefvi4g3.bkt.clouddn.com/ALC-12-1.png)

下载had-tools,将codec复制到had-tools目录，打开终端，cd到此目录，输入 ./widget_dump.sh codec.txt 回车，此处codec.txt要根据你的codec名字来，回车可以看到以下输出：

![ALC-13](http://ovefvi4g3.bkt.clouddn.com/ALC-13-1.png)

找到nid = 0x19和nid = 0x1a,这里我的19为line in，1a为mic in，记录下最后两位，我的是04和24就这么改

![ALC-14](http://ovefvi4g3.bkt.clouddn.com/ALC-14-1.png)

保存退出，双击按照AppleALC的编译方法编译这个

![ALC-15](http://ovefvi4g3.bkt.clouddn.com/ALC-15-1.png)

然后将生成的ALCPlugFix替换alc_fix中的ALCPlugFix，终端cd到alc_fix目录，执行 ./install.sh耳机就可以自动切换了，三节点的朋友运气好的话杂音应该也解决了，这时插入耳机在执行 ./widget_dump.sh codec.txt，就可以发现之前的19和1a后面的数据反过来了

![ALC-16](http://ovefvi4g3.bkt.clouddn.com/ALC-16-1.png)

如果不行重启一次应该就好了。
