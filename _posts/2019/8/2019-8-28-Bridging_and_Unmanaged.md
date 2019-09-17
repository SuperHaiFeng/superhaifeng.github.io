---
layout: post
title: "Bridging和Unmanaged"
data: 2019-8-28
subtitle: 
image: https://raw.githubusercontent.com/SuperHaiFeng/superhaifeng.github.io/master/assets/TitleImg/foundation.jpg
optimized_image: https://raw.githubusercontent.com/SuperHaiFeng/superhaifeng.github.io/master/assets/TitleImg/foundation.jpg
description: 这篇要谈论的是 Core Foundation
category: foundation
tags:
   - Cocoa
   - Foundation
   - ARC
author: Maco
paginate: true
---


有经验的读者看到这章的标题就能知道我们要谈论的是 Core Foundation。在 Swift 中对于 Core Foundation (以及其他一系列 Core 开头的框架) 在内存管理进行了一系列简化，大大降低了与这些 Core Foundation (以下简称 CF ) API 打交道的复杂程度。

首先值得一提的是对于 Cocoa 中 Toll-Free Bridging 的处理。Cocoa 框架中的大部分 `NS` 开头的类其实在 CF 中都有对应的类型存在，可以说 `NS` 只是对 CF 在更高层面的一个封装。比如 `NSURL` 和它在 CF 中的 `CFURLRef` 内存结构其实是同样的，而 `NSString` 则对应着 `CFStringRef`。

因为在 Objective-C 中 ARC 负责的只是 `NSObject` 的自动引用计数，因此对于 CF 对象无法进行内存管理。我们在把对象在 `NS` 和 `CF` 之间进行转换时，需要向编译器说明是否需要转移内存的管理权。对于不涉及到内存管理转换的情况，在 Objective-C 中我们就直接在转换的时候加上 `__bridge` 来进行说明，表示内存管理权不变。例如有一个 API 需要 `CFURLRef`，而我们有一个 ARC 管理的 `NSURL`对象的话，这样来完成类型转换：

```objective-c
NSURL *fileURL = [NSURL URLWithString:@"SomeURL"];
SystemSoundID theSoundID;
//OSStatus AudioServicesCreateSystemSoundID(CFURLRef inFileURL,
//                             SystemSoundID *outSystemSoundID);
OSStatus error = AudioServicesCreateSystemSoundID(
(__bridge CFURLRef)fileURL,
&theSoundID);
```

而在 Swift 中，这样的转换可以直接省掉了，上面的代码可以写为下面的形式，简单了许多：

```swift
import AudioToolbox
let fileURL = NSURL(string: "SomeURL")
var theSoundID: SystemSoundID = 0
//AudioServicesCreateSystemSoundID(inFileURL: CFURL,
//        _ outSystemSoundID: UnsafeMutablePointer<SystemSoundID>) -> OSStatus
AudioServicesCreateSystemSoundID(fileURL!, &theSoundID)
```

细心的读者可能会发现在 Objective-C 中类型的名字是 `CFURLRef`，而到了 Swift 里成了 `CFURL`。`CFURLRef` 在 Swift 中是被 typealias 到 `CFURL` 上的，其实不仅是 URL，其他的各类 CF 类型都进行了类似的处理。这主要是为了减少 API 的迷惑：现在这些 `CF` 类型的行为更接近于 ARC 管理下的对象，因此去掉 Ref 更能表现出这一特性。

另外在 Objective-C 时代 ARC 不能处理的一个问题是 CF 类型的创建和释放。虽然不能自动化，但是遵循命名规则来处理的话还是比较简单的：对于 CF 系的 API，如果 API 的名字中含有 `Create`，`Copy` 或者 `Retain` 的话，在使用完成后，我们需要调用 `CFRelease` 来进行释放。

不过 Swift 中这条规则已成明日黄花。既然我们有了明确的规则，那为什么还要一次一次不厌其烦地手动去写 `Release` 呢？基于这种想法，Swift 中我们不再需要显式地去释放带有这些关键字的内容了 (事实上，含有 `CFRelease` 的代码甚至无法通过编译)。也就是说，CF 现在也在 ARC 的管辖范围之内了。其实背后的机理一点都不复杂，只不过在合适的地方加上了像 `CF_RETURNS_RETAINED` 和 `CF_RETURNS_NOT_RETAINED` 这样的标注。

但是有一点例外，那就是对于非系统的 CF API (比如你自己写的或者是第三方的)，因为并没有强制机制要求它们一定遵照 Cocoa 的命名规范，所以贸然进行自动内存管理是不可行的。如果你没有明确地使用上面的标注来指明内存管理的方式的话，将这些返回 CF 对象的 API 导入 Swift 时，它们的类型会被对对应为 `Unmanaged<T>`。

这意味着在使用时我们需要手动进行内存管理，一般来说会使用得到的 `Unmanaged` 对象的 `takeUnretainedValue` 或者 `takeRetainedValue` 从中取出需要的 CF 对象，并同时处理引用计数。`takeUnretainedValue` 将保持原来的引用计数不变，在你明白你没有义务去释放原来的内存时，应该使用这个方法。而如果你需要释放得到的 CF 的对象的内存时，应该使用 `takeRetainedValue` 来让引用计数加一，然后在使用完后对原来的 `Unmanaged` 进行手动释放。为了能手动操作 `Unmanaged` 的引用计数，`Unmanaged` 中还提供了 `retain`，`release` 和 `autorelease` 这样的 "老朋友" 供我们使用。一般来说使用起来是这样的 (当然这些 API 都是我虚构的)：

```swift
// CFGetSomething() -> Unmanaged<Something>
// CFCreateSomething() -> Unmanaged<Something>
// 两者都没有进行标注，Create 中进行了创建
let unmanaged = CFGetSomething()
let something = unmanaged.takeUnretainedValue()
// something 的类型是 Something，直接使用就可以了
let unmanaged = CFCreateSomething()
let something = unmanaged.takeRetainedValue()
// 使用 something
//  因为在取值时 retain 了，使用完成后进行 release
unmanaged.release()
```

切记，这些只有在没有标注的极少数情况下才会用到，如果你只是调用系统的 CF API，而不会去写自己的 CF API 的话，是没有必要关心这些的。