---

layout: post
title: "OC与Swift混编之project-Swift.h"
data: 2019-9-6
desc: swift开源那么久了，大家肯定了解过并且使用过swift，使用oc开发那么久，项目比较大的情况下
image: https://raw.githubusercontent.com/SuperHaiFeng/superhaifeng.github.io/master/assets/TitleImg/oc-swift-title.png
optimized_image: 
description: 学习和了解逆向工程，可以帮助我们分析竞品和自己喜欢的APP的开发架构和某些功能的大体实现思路
category: swift

---

## iOS逆向解密

学习和了解逆向工程，可以帮助我们分析竞品和自己喜欢的APP的开发架构和某些功能的大体实现思路，也可以自己手动对其它APP大刀阔斧进行二次加工，满足自己的需求。

### Mac远程登录iPhone

iOS和Mac OS X都是基于Darwin（苹果的一个基于Unix的开源系统内核），所以iOS中同样支持终端的命令行操作。	

在逆向工程中，我们经常会通过命令行来操纵iPhone。为了能够让Mac终端中的命令行能作用在iPhone上，我们得让Mac和iPhone建立连接。连接有两种方式：`wifi连接`和`usb连接`。

> *先在越狱软件上安装ssh插件OpenSSH ,命令行下和应用交互的插件Cycript*	
>
> *让越狱手机和mac电脑在同一个局域网下(为了能够通过ssh服务从mac电脑访问手机)*
>
> *在mac的命令行终端 通过ssh服务登录手机 输入*`ssh root@手机ip`*。默认情况下的root密码是alpine。root密码可以自己修改。*
>
> *然后在手机上运行程序，在mac终端上利用ps -A 查看手机当前运行的进程，找到进程id后便可以利用cycript进行一些列操作。例如：进入当前运行着的微信进程的cycript状态*`cycript -p WeChat`

采用`wifi连接`有时候会出现卡顿延迟的现象，所以我通常采用`usb连接`。

> *Mac上有个服务程序usbmuxd（它会开机自动启动），可以将Mac的数据通过USB传输到iPhone*
>
> *我使用了两个脚本进行登录：*
>
> * `python ~/iOS/tcprelay.py -t 22:10010`*进行端口的映射*
>
> * `ssh -p 10010 root@localhost` *usb的登录*
>
>   ​

### Cycript的使用

Cycript是Objective-C++、ES6（JavaScript）、Java等语法的混合物，可以用来探索、修改、调试正在运行的Mac\iOS APP。官网：[http://www.cycript.org](http://www.cycript.org/)

比如一些简单的使用：  

```objective-c
// 微信进程

cycript -p WeChat

// 获得沙盒路径

NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES)[0]

// 打印当前页面view的层级

UIApp.keyWindow.recursiveDescription().toString()

```

主要搭配`Reveal`使用，从`Reveal`中获得某个界面或者`view`所属的类或控制器，然后拿到该类或控制器利用cycript进行调试。比如，知道了一个`view`对应的类为`testView`,想把该`view`从当前界面移除，达到不显示的效果：

```objective-c
[testView removeFromSuperview];
```



### 代码Hook分析

如果要逆向App的某个功能少不了代码的分析。

1. 通过上面的分析，找到某个`view`对应的类后，就需要导出该类对应的头文件进行具体的分析了。		

2. 首先找到App的二进制文件（Mach-O类型），（使用iFunBox把该文件导出到Mac上）然后使用class-dump工具导出其中的所有头文件，这些头文件中可以看到其中的属性和方法。`class-dump  -H  Mach-O文件路径  -o  头文件存放目录`

3. 如果要查看`Mach-O`文件完整信息，建议用`MachOView`。`otool -l`打印所有的 `Load Commands`，建议搭配`grep`进行正则过滤。`otool -L` 可以查看使用的库文件。

4. 头文件分析完毕后，就可以利用`theos`进行越越代码的开发了，编译生成Tweak插件(`deb`格式)。

   > *利用*`nic.pl`*指令，选择*`iphone/tweak`*，创建一个tweak工程。*
   >
   > *在这个tweak工程中编辑*`Tweak.xm`*文件，编写自己的越狱代码。*
   >
   > *开发完成后利用*`make package`*打包和*`make install`*安装到手机。重启应用，你会发现对应的功能已经根据hook的代码改变了。*
   >
   > *原理：iOS在越狱后，会默认安装一个名叫*`mobilesubstrate`*的动态库，它的作用是提供一个系统级的入侵管道，所有的*`tweak`*都可以依赖它来进行开发。在目标程序启动时根据规则把指定目录的第三方的动态库加载进去，第三方的动态库也就是我们写的破解程序，从而达到修改内存中代码逻辑的目的。*
   >
   > *利用*`nic.pl`*指令，选择*`iphone/tweak`*，创建一个tweak工程。


5. 有时候想看某个类中的某个方法的实现以及调用逻辑，就需要用到`Hopper Disassembler`工具。

#### theos的常用语法

- %hook ,%end : hook一个类的开始和结束
- %log：打印方法调用详情
- HBDebugLog：跟NSLog类似
- %new：添加一个新的方法的时候使用
- %orig：函数原来的代码逻辑
- %ctor：在加载动态库时调用
- logify.pl：可以将一个头文件快速转换成已经包含打印信息的xm文件
- 如果有额外的资源文件（比如图片），放到项目的layout文件夹中，对应着手机的根路径

### 砸壳(脱壳)

如果使用越狱手机直接从`pp助手`下载下来的部分应用免去了我们自己脱壳的过程。但是如果是从App Store下载下来的应用，App Store已经为该应用进行了加密，再使用`class-dump`是无法导出头文件的，这是时候就需要对APP进行脱壳操作了。

脱壳工具有两种，`Clutch` 和 `dumpdecrypted `

`Clutch` : 	

> *在Mac终端登陆到iPhone后，利用Clutch脱壳*
>
> `Clutch -i`	*列举手机中已安装的应用中加密的应用。*
>
> `Clutch -d  应用bundleid` *对加密的应用脱壳，脱壳成功后会生产新的*`Match-O`*文件。对这个新的文件进行*`class-dump`*操作即可。*

有时候使用`Clutch`脱壳，会出现失败的情况，比如脱壳微信的时候就会出现错误。这个时候就需要使用`dumpdecrypted `：	

> *终端进入*`dumpdecrypted.dylib`*所在的目录* `var/root`
>
> *使用环境变量* `DYLD_INSERT_LIBRARIES` *将* `dylib` *注入到需要脱壳的可执行文件（可执行文件路径可以通过*`ps -A`*查看获取）*
>
> *执行命令* `DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib 可执行文件路径` *即可完成脱壳操作。*

### 结语

了解以上逆向的流程后，你可以实现一些有趣的功能，比如:视频客户端去广告，修改微信运动步数，防止微信消息测回，微信自动抢红包等功能。同时，也会在自己客户端的开发过程中更注重信息的安全保护。研究逆向，一定要善于利用各种工具，并且做好不断失败的准备，愈挫愈勇，终会成功。



### 文件

分享的ppt在[这里](../files/hacking)，包含了逆向所需要的工具列表。

 