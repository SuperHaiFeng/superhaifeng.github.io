---

layout: post
title: "手机触摸屏实现原理"
date: 2019-10-8
desc: 这篇文章旨在介绍我们常用的智能手机触摸屏的实现原理
image: https://raw.githubusercontent.com/SuperHaiFeng/superhaifeng.github.io/master/assets/TitleImg/oc-swift-title.png
optimized_image: 
description: 这篇文章旨在介绍我们常用的智能手机触摸屏的实现原理
category: touch

---

![](../../../../assets/TitleImg/mac.png)

对于屏幕，在生活中随处可见，电脑、电视、手机等等，我们今天要介绍一下平时用的手机触摸屏或者电脑触摸屏的基本原理。

触摸屏的主要三大种类有电阻技术触摸屏、表面声波技术触摸屏、电容技术触摸屏，没一类触摸屏都有自己的优缺点，比如我们使用的Kindle，之前的Mobile phone等基本都是使用的电阻技术触摸屏，我们现在使用的大多数智能手机，触摸屏电脑，iPad等基本都是使用的电容技术触摸屏，银行自助设备、触摸一体机等基本都是使用的表面声波技术触摸屏，今天我们主要介绍电容技术触摸屏。

在介绍之前我们先回顾一下高中学过的物理知识：电流和电容。 

### 一、电流

[科学](https://baike.baidu.com/item/%E7%A7%91%E5%AD%A6/10406)上把单位时间里通过导体任一[横截面](https://baike.baidu.com/item/%E6%A8%AA%E6%88%AA%E9%9D%A2/3338627)的[电量](https://baike.baidu.com/item/%E7%94%B5%E9%87%8F/10950375)叫做[电流强度](https://baike.baidu.com/item/%E7%94%B5%E6%B5%81%E5%BC%BA%E5%BA%A6/5333886)，简称电流，电流符号为I，单位是安培（*A*），简称“安”（[安德烈·玛丽·安培](https://baike.baidu.com/item/%E5%AE%89%E5%BE%B7%E7%83%88%C2%B7%E7%8E%9B%E4%B8%BD%C2%B7%E5%AE%89%E5%9F%B9/6806706)，1775年—1836年，[法国](https://baike.baidu.com/item/%E6%B3%95%E5%9B%BD/1173384)物理学家、化学家，在[电磁](https://baike.baidu.com/item/%E7%94%B5%E7%A3%81)作用方面的研究成就卓著，对数学和物理也有贡献。电流的国际单位安培即以其姓氏命名）。[导体](https://baike.baidu.com/item/%E5%AF%BC%E4%BD%93/1017277)中的[自由电荷](https://baike.baidu.com/item/%E8%87%AA%E7%94%B1%E7%94%B5%E8%8D%B7/6393682)在[电场力](https://baike.baidu.com/item/%E7%94%B5%E5%9C%BA%E5%8A%9B/1845623)的作用下做有规则的定向运动就形成了 电流。

我们知道，在生活中一般有两种电流，一种是直流，一种是交流，现在我们回顾一下什么是直流电和交流电。

**直流电**

直流（DC）原来的英文名称是galvanic current，也称原义是指[电荷](https://zh.wikipedia.org/wiki/%E9%9B%BB%E8%8D%B7)的单向流动，一般是由像[电池](https://zh.wikipedia.org/wiki/%E9%9B%BB%E6%B1%A0)、[太阳能电池](https://zh.wikipedia.org/wiki/%E5%A4%AA%E9%98%B3%E8%83%BD%E7%94%B5%E6%B1%A0)等设备产生。直流电流可以在[导体](https://zh.wikipedia.org/wiki/%E5%B0%8E%E9%AB%94)（例如电线）中流动，也可以在[半导体](https://zh.wikipedia.org/wiki/%E5%8D%8A%E5%B0%8E%E9%AB%94)、[绝缘体](https://zh.wikipedia.org/wiki/%E7%B5%95%E7%B7%A3%E9%AB%94)中流动，甚至在真空也可以以[离子束](https://zh.wikipedia.org/wiki/%E9%99%B0%E6%A5%B5%E5%B0%84%E7%B7%9A)的方式流动。在直流电中，电子以固定的方向流动。
![](../../../../assets/touch-screen/DC.png)
第一个商业化的电力传输由[汤玛斯·爱迪生](https://zh.wikipedia.org/wiki/%E6%B9%AF%E7%91%AA%E6%96%AF%C2%B7%E6%84%9B%E8%BF%AA%E7%94%9F)在十九世纪后期开发，使用110伏特的直流电。然而由于在传输和电压转换的优势差异，今天几乎所有的电力分配为[交流电](https://zh.wikipedia.org/wiki/%E4%BA%A4%E6%B5%81%E7%94%B5)。在20世纪50年代中期，曾经发展过[超高压直流电系统](https://zh.wikipedia.org/wiki/HVDC)，现在该技术是在远程及水下电力传输上，除了高压交流电以外的另一种选项然而并不常见。但是特种应用要求上，如一些[第三轨](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%B8%89%E8%BB%8C)或[架空电车线](https://zh.wikipedia.org/wiki/%E6%9E%B6%E7%A9%BA%E9%9B%BB%E8%BB%8A%E7%B7%9A)的[铁路](https://zh.wikipedia.org/wiki/%E9%93%81%E8%B7%AF)电力系统还是用直流电，交流电被分配到一个[变电站](https://zh.wikipedia.org/wiki/%E5%8F%98%E7%94%B5%E7%AB%99)利用一个整流器转换为直流电。

![](../../../../assets/touch-screen/Brush_central_power_station_dynamos_New_York_1881.jpg)

<center>1880年的爱迪生直流发电机，后来连接了一个3.2公里长的输电线</center>



**交流电**

交流（AC）原义是指[电荷](https://zh.wikipedia.org/wiki/%E9%9B%BB%E8%8D%B7)的运动会周期性的变换方向，和直流不同，直流电流的电荷只会单方向流动。一般商业、家用及工业用电多半是交流电，例如一般插座提供的电就是交流电。最常见的交流电波形是[正弦波](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%BC%A6%E6%B3%A2)，但在特殊应用中也会出现其他的波形，像[三角波](https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E6%B3%A2)或[方波](https://zh.wikipedia.org/wiki/%E6%96%B9%E6%B3%A2)。像[调幅广播](https://zh.wikipedia.org/wiki/%E8%AA%BF%E5%B9%85%E5%BB%A3%E6%92%AD)及[调频广播](https://zh.wikipedia.org/wiki/%E8%B0%83%E9%A2%91%E5%B9%BF%E6%92%AD)的讯号也是交流的例子之一，其目的是在利用[调变](https://zh.wikipedia.org/wiki/%E8%AA%BF%E8%AE%8A)技术，在交流讯号中加入要传递的讯号后传递，而接收端可以再还原为原始的讯号。

当发现了[电磁感应](https://zh.wikipedia.org/wiki/%E7%94%B5%E7%A3%81%E6%84%9F%E5%BA%94)后，产生交流电流的方法就被知晓。早期的成品由英国人[麦可·法拉第](https://zh.wikipedia.org/wiki/%E9%BA%A6%E5%8F%AF%C2%B7%E6%B3%95%E6%8B%89%E7%AC%AC)（Michael Faraday）与法国人[波利特·皮克西](https://zh.wikipedia.org/w/index.php?title=%E6%B3%A2%E5%88%A9%E7%89%B9%C2%B7%E7%9A%AE%E5%85%8B%E8%A5%BF&action=edit&redlink=1)（Hippolyte Pixii）等人开发出来。

1882年，英国电工[詹姆斯·戈登](https://zh.wikipedia.org/w/index.php?title=%E8%A9%B9%E5%A7%86%E6%96%AF%C2%B7%E6%88%88%E7%99%BB&action=edit&redlink=1)建造大型双相交流发电机。[开尔文勋爵](https://zh.wikipedia.org/wiki/%E5%BC%80%E5%B0%94%E6%96%87%E5%8B%8B%E7%88%B5)与[塞巴斯蒂安·费兰蒂](https://zh.wikipedia.org/w/index.php?title=%E5%A1%9E%E5%B7%B4%E6%96%AF%E8%92%82%E5%AE%89%C2%B7%E8%B4%B9%E5%85%B0%E8%92%82&action=edit&redlink=1)（Sebastian Ziani de Ferranti）开发早期交流发电机，频率介于100赫兹至300赫兹之间。

1891年，[尼古拉·特斯拉](https://zh.wikipedia.org/wiki/%E5%B0%BC%E5%8F%A4%E6%8B%89%C2%B7%E7%89%B9%E6%96%AF%E6%8B%89)取得了“高频率”（15,000赫兹）交流发电机的专利。

1891年后，多相交流发电机被用来供应电流，此后的交流发电机的交流电流频率通常设计在16赫兹至100赫兹间，搭配弧光灯、白炽灯或电动机使用。

根据电磁感应定律，当导体周围的磁场发生变化，感应电流在导体中产生。通常情况下，旋转磁体称为[转子](https://zh.wikipedia.org/wiki/%E8%BD%AC%E5%AD%90)，导体绕在铁芯上的线圈内的固定组，称为[定子](https://zh.wikipedia.org/wiki/%E5%AE%9A%E5%AD%90)，当其跨越磁场时，便产生电流。产生交流电的基本机械称为[交流发电机](https://zh.wikipedia.org/wiki/%E4%BA%A4%E6%B5%81%E7%99%BC%E9%9B%BB%E6%A9%9F)。

我们国家生活用电使用的是220V的交流电，但是其他国家不一定，比如美国一般使用的是110V，朝鲜、日本等民用是100V，澳大利亚等一些国家使用240V，所以每个国家有每个国家的制定标准，所以生产出来的电器产品也是根据国家的标准来制定的。

物理上规定电流的方向是正电荷定向流动的方向（即正电荷定向流动的速度的正方向或负电荷定向流动的速度的反方向），电流运动方向和电子运动方向相反。

我们了解了电流，下面我们再回顾一下电容器。

### 二、电容器

电容器（capacitor）是将[电能](https://zh.wikipedia.org/wiki/%E9%9B%BB%E8%83%BD)储存在[电场](https://zh.wikipedia.org/wiki/%E9%9B%BB%E5%A0%B4)中的[被动](https://zh.wikipedia.org/wiki/%E8%A2%AB%E5%8B%95%E5%85%83%E4%BB%B6)[电子元件](https://zh.wikipedia.org/wiki/%E9%9B%BB%E5%AD%90%E5%85%83%E4%BB%B6)。电容器的储能特性可以用[电容](https://zh.wikipedia.org/wiki/%E9%9B%BB%E5%AE%B9)表示。在[电路](https://zh.wikipedia.org/wiki/%E9%9B%BB%E8%B7%AF)中邻近的[导体](https://zh.wikipedia.org/wiki/%E5%B0%8E%E9%AB%94)之间即存在电容，而电容器是为了增加电路中的[电容量](https://zh.wikipedia.org/wiki/%E9%9B%BB%E5%AE%B9%E9%87%8F)而加入的电子元件。

历史上第一个有留下记录的电容器由[克拉斯特主教](https://zh.wikipedia.org/wiki/%E5%85%8B%E6%8B%89%E6%96%AF%E7%89%B9%E4%B8%BB%E6%95%99)在1745年10月所发明，是一个内外层均镀有金属膜的玻璃瓶，玻璃瓶内有一金属杆，一端和内层的金属膜连结，另一端则连结一金属球体。借由在二层金属膜中利用玻璃作为绝缘的方式，克拉斯特主教让[电荷密度](https://zh.wikipedia.org/wiki/%E9%9B%BB%E8%8D%B7%E5%AF%86%E5%BA%A6)出现明显的提升。

1746年1月，荷兰物理学家[彼得·范·穆森布罗克](https://zh.wikipedia.org/w/index.php?title=%E5%BD%BC%E5%BE%97%C2%B7%E8%8C%83%C2%B7%E7%A9%86%E6%A3%AE%E5%B8%83%E7%BD%97%E5%85%8B&action=edit&redlink=1)也独立发明了构造非常类似的电容器，当时克拉斯特主教的发明尚未广为人知。由于马森布鲁克当时在[莱顿大学](https://zh.wikipedia.org/wiki/%E8%90%8A%E9%A0%93%E5%A4%A7%E5%AD%B8)任教，因此将其命名为[莱顿瓶](https://zh.wikipedia.org/wiki/%E8%8E%B1%E9%A1%BF%E7%93%B6)。

当时人们认为，电荷是储存在莱顿瓶中的水里；但美国科学家[富兰克林](https://zh.wikipedia.org/wiki/%E7%8F%AD%E5%82%91%E6%98%8E%C2%B7%E5%AF%8C%E8%98%AD%E5%85%8B%E6%9E%97)研究莱顿瓶，证明其电荷是储存在玻璃上，并非储存在莱顿瓶中的水里。

电容器包括两个导电物质（电极），两个电极储存的电荷大小相等，符号相反，电极本身是导体，两个电极之间由介电质的绝缘体隔开，电极的金属片通常用的是铝片或是铝箔，若用氧化铝来做介质的就是电解电容器。电荷会储存在电极表面，靠近介电质的部分。由于二个电极储存的电荷大小相等，符号相反，因此电容器中始终保持为电中性。

![](../../../../assets/touch-screen/Capacitor.png)

如上图所示：这是电容器充电过程，右边是电池，上面是正极，下面是负极，所以正极的电子往上流，负极的电子往下流，所以会导致电容器的下面会带负电荷，上面带正电荷，直到电容器中的电压和电池的电压相等的时候就不会再进行充电了，这个线路图中就不会再有电流流动，因为电池是直流电，所以这个只是一个充电过程。假如我们把直流电换成交流电，电容器就会产生充电和放电的操作，交流电相当于我们周期性的把电池来回颠倒，一会上面正极，一会上面负极，这样的话，电容器上下两极的电荷就会一直流动，电容器就会保持充放电的过程，所以电路中就会保持一致有电流流过。

好了，我们理解了电流和电容器，接下来我们步入正题，电容技术触摸屏。

### 三、电容技术触摸屏

电容技术触摸屏是利用我们人体的电流感应来进行工作的，我们先来看一下触摸屏的基本组成。

![](../../../../assets/touch-screen/Capacitor-screen.jpg)

电容式触摸屏由一个模拟感应器和一个双向智能控制器组成 。模拟感应器是一块四层复合玻璃屏 ,玻璃屏的内表面和夹层各有一层聚酯材料导电涂层 ,最外是只有 0. 0015mm 厚的矽土玻璃 ,形成坚实耐用的保护层 。夹层作为工作面 ,四个角上各引出一个电极内层作为屏蔽层用以保证良好的工作环境。电容式触摸屏在触摸屏四边均镀上狭长的电极，在导电体内形成一个低电压交流电场，触摸屏工作时 ,感应器边缘的电极产生分布的均匀电压场 ,由于人体电场的存在 ,触摸屏幕时 ,手指和触摸屏的工作面之间会形成一个耦合电容 ,因为工作面上接有高频信号 ,于是手指吸走一个很小的电流 ,然后分别从触摸屏四个角上的电极中流出 。从理论上讲 ,流经这四个电极的电流与手指到四角的距离成比例 ,控制器通过对这四个电流比例的精密计算 ,从而可以得出触摸点的位置 。最后 ,控制器将数字化的触摸位置数据传送给主机。

那么控制器是如何进行计算触摸点的位置的哪，如下图：

![](../../../../assets/touch-screen/calculator-point.png)

控制器先根据四个电极流出的电流大小计算出触摸点到四个电极的距离，然后以这个距离为半径进行画圆，会画成四个圆，这四个圆的交点就是触摸点的位置。图中的灰色圆点就是触摸点，四条灰线是到电极的距离，红色的圆是以灰线为半径画的圆，我们看到，交点正好在我们的触摸点。

电容屏要实现多点触控，靠的就是增加互电容的电极，简单地说，就是将屏幕分块，在每一个区域里设置一组互电容模块都是独立工作，所以电容屏就可以独立检测到各区域的触控情况，进行处理后，简单地实现多点触控。

由于电容随接触面积、介质的介电的不同而变化，故其稳定性较差，往往会产生漂移现象，比如我们将整个手掌都放在触摸屏上移动，就会出现屏幕来回漂移的现象，或者手指上沾上水，触摸触摸屏也会出现这个现象，主要原因是控制器计算触摸点的时候会有多个点的计算，计算会出现误差，所以会导致漂移现象。正常情况下是没有问题的，一般能达到99%的准确率，延时在3ms左右。

### 四、apple pencil

写到这里，我突然想说几句最近两年apple出的一款apple pencil，那么apple pencil是怎样的一个工作原理哪，为什么除了手也可以使用其他的工具操作屏幕哪。

其实之前别的公司就出过一些电容笔，专门来操作电容屏的，比如微软出的surface pen，但是这些电容笔需要手拿着来操作屏幕，那么apple pencil不需要手拿也可以操作屏幕，我们来看看它的基本原理。  

**触控定位精度**

Apple Pencil其实并不是“电容笔”，是主动在笔尖发射电磁波的。iPad端的支持Pencil的触控芯片也是特殊定制的，在扫描普通的触控信号的同时，接受到Pencil的特殊频段，这个可以拿频谱探棒扫描到的。
电容触摸屏对于直流电来说是绝缘的，但是对于交流信号是可以理解是微弱导电的。
\- 更惊人的是，笔尖不是发射一个固定频段的哦，而是同时发射多个频段的信号的哦，利用交叠扫描和数学分析提高定位精度。
\- 更更惊人的是，适配不同iPad 频段是不一样的哦，是每台每台校正的哦，用来规避不同LCD面板的微弱电磁噪声从而提高信噪比
所以结果就是，Apple Pencil 可以实现接近像素级的定位精度，至少10倍于电容笔。
如果你考虑到Apple一直在触控密度和扫描频率都是坚持高业界一个档次的，Apple Pencil还有240Hz扫描频率

**传感器**

笔尖是有压力传感器感受用力的，而iPad屏幕是不支持force touch的。 
另外Apple Pencil里还有9轴陀螺仪，用来和iPad的陀螺仪的差值计算笔尖的倾斜角度，实现毛笔的笔触效果。。
\- 更惊人的是，如果看过拆解，里面不是1颗陀螺仪哦，而是在不同位置装了2颗。可能外行同学不太明白2颗有什么惊人的，如果有做硬件设计或者硬件产品经理同学可以想一下2颗陀螺仪能用来做什么。
Apple 连你倒立拿着笔写字的情况都考虑了，还可以实现类似转笔过来用笔帽做橡皮的功能（甚至很可能Apple做了但是自己觉得效果不好又拿掉了）。。。

**算法**

最新的iOS13 有一项很不起眼的更新，Apple Pencil的延时从16ms继续提升到了8ms。
这个外行可能也觉得没啥，虽然16ms 已经是业界最最顶尖水平了，但是你可以觉得只是Apple贵，规格高，iPad Pro 120Hz现实加上Apple Pencil 240Hz 扫描频率，CPU速度也足够快，16ms（1帧）运算显示延时水到渠成。
但是8ms。。。0.5帧呀，这意味着已经违反了信息论了，触控数据还没扫描到，就已经送去显示了。
人家轻描淡写了一句“运用机器学习技术的笔迹预测”。。。

是不是感觉apple pencil技术很牛逼，如果想看apple pencil的拆解过程，可以到[这里](https://zh.ifixit.com/)

思考问题：如果人站在一个绝缘的地毯上去触摸生活中的火线，会触电吗？？

------

参考资料

[[电容式触摸屏]](https://baike.baidu.com/item/%E7%94%B5%E5%AE%B9%E5%BC%8F%E8%A7%A6%E6%91%B8%E5%B1%8F/7533199?fromtitle=%E7%94%B5%E5%AE%B9%E5%B1%8F&fromid=4907602)

[触摸屏原理及技术发展简介]

[[Apple pencil的原理是什么]](https://www.zhihu.com/question/67483519)

[[电容器]](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AE%B9%E5%99%A8)

[触摸屏技术浅谈]   *【解放军理工大学理学院  210016】*