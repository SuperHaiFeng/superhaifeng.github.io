---

layout: post
title: "从Swift桥接文件到Clang-LLVM"
data: 2019-8-29
subtitle: 
image: https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559825145/theme16_o0seet.jpg
optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825145/theme16_o0seet.jpg
description: GNU编译器套件（GNU Compiler Collection）包括C、C++、Objective-C、Fortran
category: swift
tags:
   - swift
   - Clang-LLVM
   - c++
author: Maco
paginate: true
---

### 编译器研究

- GCC
  GNU编译器套件（GNU Compiler Collection）包括C、C++、Objective-C、Fortran、Java、Ada和Go语言的前端，也包括了这些语言的库（如libstdc++、libgcj等等）

  早起的OC 程序员都感受过GCC编译程序，但是苹果为什么好好的GCC不用，自己要搞一套呢？

  1.GCC 的 Objective-C Frontend不给力：GCC的前端不是苹果提供维护的，想要添加一些语法提示等功能还得去求GCC的前端去做。

  2.GCC 插件、工具、IDE的支持薄弱：很多编译器特性没有，自动补全、代码提示、warning、静态分析等这些流程不是很给力，都是需要IDE调用底层命令完成的，结果需要以插件的形式暴露出来，这一块GCC做的不是很好。

  3.GCC 编译效率和性能不足：Apple的Clang出来以后，其编译效率是GCC的3倍，编译器性能好，编译出的文件小。

  4.Apple要收回去工具链的控制 （lldb, lld…）: Apple在早起从GCC前端到LLVM后端的编译器，到Clang-LVVM的编译器，以后后来的GDB的替换，一步一步收回对编译工具链的控制，也为Swift 的出现奠定基础。

- Three-Phase 编译器架构
  ![](../../../../assets/Clang-LLVM/three-phase.png)

  上图是最简单的三段式编译器架构。

  首先，我们看到`source` 是我们的源代码，进入编译器的前端`Frontend`；在前端完成之后，就进入优化器这一模块；优化完成之后进入后端这一模块；在这全部完成之后，根据你的架构是x86，armv7等生产机器码。

  但是会有一个问题：

  ```
  M (Language) * N (Target) = M * N (Compilers)
  ```

  就是如果你有M种语言(C、C++、Objective-C…)，N种架构(armv7、armv7s、arm64、i386、x86_64…)，那么你就有M * N中编译方式需要处理，显然是不合理的。

- **apple** 的 `Clang/Swift - LLVM` 编译器架构
  ![](../../../../assets/Clang-LLVM/Swift-LLVM.png)

  其中优化器部分(Common Optimizer)是共享的。而对于每一种语言都有其前端部分，假如新有一门语言，只需要实现该语言的前端模块；如果新出一台设备，它的架构不同，那么也只需要单独完成其后端模块即可。改动非常小，不会做重复的工作。

  下面详解：
  ![](../../../../assets/Clang-LLVM/Swift-LLVM2.png)

  蓝色的部分：C语言家族系列的前端，属于Clang部分。

  绿色的部分： Swift语言的前端，其中还包含自己的SIL中间语言和Optimzer中间语言的优化过程。

  紫色的部分： 优化阶段和后端模块统一是LLVM部分。

- 代码规模
  Clang + LLVM 代码模块总共有400W行代码，其中主体部分是C++写的，大概有235W行。如果将所有的target，lib等文件编译出来，大概有近20G的大小：

  ![](../../../../assets/Clang-LLVM/Clang+LLVM.png)

  对比Swift Frontend 代码规模，就少很多，只有43W行左右。可能在后端，比如优化器策略，生成机器码部分就有很多代码：
  ![](../../../../assets/Clang-LLVM/Swift-fronted.png)

- Clang命令Clang在概念上是编译器前端，同时，在命令行中也作为一个“黑盒”的 Driver;

  它封装了编译管线、前端命令、LLVM命令、Toolchain命令等，即一个Clang走天下；

  方便从GCC迁移过来。

  当我们点击run命令以后，如下图：
  ![](../../../../assets/Clang-LLVM/Run-Clang.png)

  就是我们在**build setting**中的一些设置，组装成命令，下面可以看到是一个 oc文件在arc环境下的编译过程：
  ![](../../../../assets/Clang-LLVM/Run-Clang2.png)

- 拆解编译过程

  ```
  import <Foundation/Foundation.h>
  
  int main() {
      @autoreleasepool {
          id obj = [NSObject new];
          NSLog(@"Hellow world: %@", obj);
      }
  }
  ```

1. **Preprocess - 预处理**import 头文件，include头文件等 
   macro宏展开 
   处理’#’大头的预处理指令，如 #if，#elseif等

   终端输入：

   ```
   $ clang -E main.m
   ```

   只会做预处理步骤，不往后面走，如下

   ![](../../../../assets/Clang-LLVM/Clang-main.png)

   可以看到一个头文件要导入很多行代码，这里就要说到pch文件。本身Apple给出这个文件，是让我们放入Foundation或者UIKit等这些根本不会变的库，优化编译过程，但是开发者却各种宏，各种头文件导入，导致编译速度很慢。以至于后来苹果删除了这个文件，只能开发者自己创建。但是苹果提供modules这个概念，可以通过以下命令打开：

   ```
   $ clang -E -fmodules main.m
   ```

   默认把一些文件打包成库文件, 在**build setting**中默认打开的，我们可以用@import Foundation:
   ![](../../../../assets/Clang-LLVM/Clang-main2.png)

2. **Lexical Analysis - 词法分析**

   词法分析，也作Lex 或者 Tokenization 
   将预处理过得代码文本转化为Token流 
   不会校验语义

   可以在终端输入以下命令：

   ```
   $ clang -fmodules -fayntax-only -Xclang -dump-tokens main.m
   ```

   ![](../../../../assets/Clang-LLVM/Lexical-Analysis.png)

3. **Analysis - 语法分析**
   语法分析，在Clang中有Parser和Sema两个模块配合完成，验证语法是否正确，并给出正确的提示。这就是Clang标榜GCC，自己的语法提示友好的体现。

   根据当前的语法，生成语意节点，并将所有节点组合成抽象语法书(**AST**)

   输入命令：

   ```
   $ clang -fmodules -fsyntax-only -Xclang -ast-dump main.m
   ```

   ![](../../../../assets/Clang-LLVM/Analysis.png)

   可以通过语法树，反写回源码，如下图：

   ![](../../../../assets/Clang-LLVM/AST.png)

4. **Static Analysis - 静态分析**(不是必须的)

   通过语法书进行代码静态分析，找出**非语法性错误** 
   模拟代码执行路径，分析出 contro-flow graph (CFG) 
   预置了常用的 Checker

   在Xcode中如下操作可以实现：

   ![](../../../../assets/Clang-LLVM/Static-Analysis.png)

5. **CodeGen - IR 代码生成**CodeGen负责将语法树从顶至下遍历，翻译成LLVM IR，LLVMIR 是Frontend的输入，也是LLVM Backend 的输入，是前后端的桥接语言。

   与Objective-C Runtime 桥接

   ①Class / Meta Class / Protocol / Category 内存结构生成，并存放在指定 session中(如Class： _DATA, _objc_classrefs)

   ②Method / Ivar / Property 内存结构生成

   ③组成 method_list / ivar_list / property_list并填入Class

   ④Non-Fragile ABI: 为每个Ivar合成 OBJC_IVAR_$_偏移常量

   ⑤存取 Ivar的语句(ivar = 123; int a = _ivar;) 转写成base + OBJC_IVAR$_的形式

   ⑥将语法树中的 ObjCMessageExpr 翻译成相应版本的objc_msgSend，对super关键字的调用翻译成objc_msgSendSuper

   ⑦处理@synthsynthesize

   ⑧生成 block_layout 的数据结构

   ⑨变量 capture （__block/ __weak）

   10.生成_block_invoke函数

   11.ARC: 分析对象引用关系，将 objc_storeStrong/ objc_storeWeak 等ARC 代码插入

   12.将 ObjCAutoreleasePoolStmt 转译成objc_autoreleasePoolPush/Pop

   13.实现自动调用[super dealloc]

   14.为每个拥有 ivar 的Class 合成.cxx_destructor 方法来自动释放类的成员变量，代替MRC 时代下的”self.xxx = nil”

   ![](../../../../assets/Clang-LLVM/Example.png)

   终端输入：

   ```
   $ clang -S -fobjc-arc -emit-llvm main.m -o main.m1
   ```

   输入如下： 

   ![](../../../../assets/Clang-LLVM/CodeGen.png)

6. **LVVM Bitcode - 生成字节码**
   输入命令：

   ```
   $ clang -emit-llvm -c main.m -o main.bc
   ```

   ![](../../../../assets/Clang-LLVM/LVVM-Bitcode.png)

   相信大家在iOS 9之后都听过这个概念，其实就是对IR生成二进制的过程。

7. **Assemble - 生成Target相关汇编**

   终端输入：

   ```
   $ clang -S -fobjc-arc main.m -o main.s
   ```

   ![](../../../../assets/Clang-LLVM/Assemble.png)

8. **Assemble - 生成Target相关 Object(Mach-o)** 
   终端输入：

   ```
   $ clang -fmodules -c main.m -o main.o
   ```

   ![](../../../../assets/Clang-LLVM/Object(Mach-o).png)

   汇编的main.o的形式。

9. **Link 生成 Executable** 

   终端输入：

   ```
   $ clang main.m -o main
   $ ./main
   ```



   ![](../../../../assets/Clang-LLVM/Executable.png)



   ![](../../../../assets/Clang-LLVM/end.png)


   至此，我猜测可能桥接文件是在Clang阶段，将OC文件进行编译，生成语法树，然后再返成Swift能识别的类文件。

   - 我们能在Clang上做什么？

   Apple给我们留了3个接口：


1.LibClang 
功能： 
①C 的API来访问Clang的上层能力，比如获取Tokens、遍历语法树、代码补全、获取诊断信息； 
②API稳定，不受Clang源码更新影响 
③只有上层的语法树可以访问，不能获取到全部信息 
④使用原始的 C的API 
⑤脚本语言： 使用官方提供的 python binding 或开源的 node-js / ruby binding 
⑥Objective-C： 开源库 ClangKit

2.LibTooling 
①对语法树 有完全的控制权 
②可作为一个 standalone 命令单独使用，如 clang-format 
③需要使用C++且对Clang源码熟悉

3.ClangPlugin 
①对语法树有完全的控制权 
②作为插件注入到编译流程中，可以影响build和决定编译过程 
③需要使用C++且对Clang源码熟悉