---

layout: post
title: "OC与Swift混编之project-Swift.h"
data: 2019-9-6
desc: swift开源那么久了，大家肯定了解过并且使用过swift，使用oc开发那么久，项目比较大的情况下
image: https://raw.githubusercontent.com/SuperHaiFeng/superhaifeng.github.io/master/assets/TitleImg/oc-swift-title.png
optimized_image: 
description: 这篇博客为大家介绍一个iOS混编
category: swift
tags:
   - Swift
   - Objective-C
   - 桥接文件
author: Maco
paginate: true
---

swift开源那么久了，大家肯定了解过并且使用过swift，使用oc开发那么久，项目比较大的情况下，肯定不能一下全部换成swift，有的同学可能会先使用oc与swift混合编程，所以就涉及到混合编程的一些知识和文件的坑，我在这里简单介绍一下，有不足之处，可以查看下方联系我哦。

### 一、桥接文件project-Bridging-Header.h

在oc项目中创建swift文件的时候，xcode会提示是否创建这个文件，创建好以后，在build setting下的swift compler-general的objective-c bridging header下有这个文件的路径，这个路径一定要写对，否则会报错，当然这个文件也可以手动创建，创建名字要按规定，这个文件主要是进行swift调用oc中的方法和属性使用，这个文件中要import引入需要在swift中调用的oc的类名。

### 二、project-Swift.h文件

这个文件不用创建，只要使用了混编，并且创建了桥接文件，project-Swift.h文件就会自动创建，否则是没有的，这个文件的作用主要是混编时，在oc中使用swift的文件。

### 三、project-Swift.h是做什么的

我们可以定位到这个文件查看源码

![](../../../../assets/swift_img/project-Swift.png)

里面是一些宏定义和导入的一些框架，最下面是swift对应的oc代码，现在我只是创建了SwiftViewController.swift的文件，所以只@interface了这一个文件，所以如果我们有很多swift文件的时候，在这个文件中会出现对应的oc的代码，到这里我们应该已经明白了，当我们使用oc调用swift文件时，编译器会帮我们定为到这个文件，调用这个文件对应的oc的方法或者属性。

那么在swift5.0中我们在swift中写的方法和属性怎么会添加到这个文件哪，当我们创建方法或者属性的时候，需要在方法或属性前面加上@objc来进行修饰，这样会在project-Swift.h文件中自动添加上对应的oc代码：

![](../../../../assets/swift_img/project-Swift2.png)

如果我们不加@objc的话，在project-Swift.h文件中不会出现对应的oc代码，那么我们就无法使用oc调用该方法

对于 枚举类型也是使用同样的方法，如前面截图，但是对于swift特有的枚举，使用这样的方法就不行了，比如CouponResult这个枚举，他不是NSInterger类型，并且枚举中是带有参数的方法，在oc中是没有这种枚举的，所以如果在前面加@objc会报错，在oc中也是不可用的。

![](../../../../assets/swift_img/error.png)

当然如果使用枚举，在project-Swift.h同样会存在oc的代码

![](../../../../assets/swift_img/enum.png)

### 四、oc项目导入#import "project-Swift.h"报错

如果我们项目名中包含.符号，在我们引入project-Swift.h文件时会报错，比如我们的工程名叫oc.swift，只要在Build Setting->Packaging->Product Bundle Name名字为oc_swift，然后引入的时候写#import "oc_swift-Swift.h"就不会报错了，因为project-Swift.h本质上是ProductBundleName-Swift.h。

如果报错了，可以从以下方面着手检查：

- 检查项目名是否合法
- 检查项目名是否正确，不能有错误
- 检查桥接文件的路径是否正确。