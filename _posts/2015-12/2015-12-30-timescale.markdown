---
layout: post
title: verilog中timesclae的使用
data: 2015-12-30
categories: Maco
desc: timescale是Verilog HDL 中的一种时间尺度预编译指令，它用来定义模块的仿真 时的时间单位和时间精度...
---



**描述:**
timescale是Verilog HDL 中的一种时间尺度预编译指令，它用来定义模块的仿真 时的时间单位和时间精度。格式如下：

`timescale  仿真时间单位/时间精度

注意：用于说明仿真时间单位和时间精度的 数字只能是1、10、100，不能为其它的数字。而且，时间精度不能比时间单位还要大。最多两则一样大。比如：下面定义都是对的：

`timescale   1ns/1ps

`timescale   100ns/100ns

下面的定义是错的：

`timescale  1ps/1ns

时间精度就是模块仿真时间和延时的精确程序，比如：定义时间精度为10ns， 那么时序中所有的延时至多能精确到10ns，而8ns或者18ns是不可能做到的。
在编译过程中,timescale指令影响这一编译器指令后面所有模块中的时延值，直至遇到另一个timescale指令resetall指令。
在verilog中是没有默认timescale的，一个没有指定timescale的verilog模块就有可能错误的继承了前面编译模块的无效timescale参数.
下面举个简单的例子说明一下:
比如我们来获取一首歌曲的当前时间

```
//计算当前音乐时间
        NSTimeInterval time = self.player.currentTime.value / self.player.currentTime.timescale;
```
以下转载另一篇文章
当一个设计中的多个模块带有自身的timescale编译指令时将发生什么?在这种情况下，模拟器总是定位在所有模块的最小时延精度上，并且所有时延都相应的换算为最小时延精度例如:

```
timescale 1ns/100ps
MODULE AndFunc(Z,A,B);
OUTPUT Z;
input A,B;
and #(5.22,6.17) AL(Z,A,B);
endMODULE

timescale 10ns/1ns
MODULE TB;
reg PutA,PutB;
WHRE GetO;
initial
begin
PutA = 0;
PutB = 0;
#5.21 PutB = 1;
#10.4 PutA = 1;
#15 PutB = 0;
end
AndFunc AF1(GetO,PutA,PutB);
endMODULE
```

这个例子中，每个模块都有自身的timescale编译器指令。timescale编译器指令第一次应用于时延。因此在第一个模块中，5.22对应5.2ns，6.17对应6.2ns;在第二个模块中5.21对应52ns，10.4对应104ns，15对应150ns，如果仿真模块TB，设计中的所有模块最小时间精度为100ps。因此，所有延迟（特别是模块TB中的延迟）将换算成精度为100ps。延迟52ns现在对应520*100ps，104对应1040*100ps，150对应1500*100ps，更重要的是，仿真使用100ps为时间精度。如果仿真模块AndFunc，由于模块TB不是模块AndFunc的子模块，模块TB中的timescale程序指令将不再有效。