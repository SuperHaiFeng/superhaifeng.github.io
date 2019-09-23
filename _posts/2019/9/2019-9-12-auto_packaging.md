---

layout: post
title: "iOS自动化打包"
data: 2019-9-12
desc: 频繁的手动打包是一项耗时耗力的工程，而且是一项重复性的劳动，因此实现打包的自动化是非常必要
image: https://raw.githubusercontent.com/SuperHaiFeng/superhaifeng.github.io/master/assets/TitleImg/oc-swift-title.png
optimized_image: 
description: 频繁的手动打包是一项耗时耗力的工程，而且是一项重复性的劳动
category: swift

---

## 自动化打包

频繁的手动打包是一项耗时耗力的工程，而且是一项重复性的劳动，因此实现打包的自动化是非常必要的。通过自动化打包可以实现一键打包，并上传到fir或蒲公英等第三方平台。

## xcodebuild

当我们通过`Archive`手动打包的时候，`Xcode`本身是通过调用`xcodebuild`命令来实现打包的过程。

`xcodebuild`是苹果提供的用于打包项目或者工程的命令，可以通过`man xcodebuild`命令查看它的介绍。

> NAME		
> 
> xcodebuild -- build Xcode projects and workspaces
> 
> DESCRIPTION
> 
> xcodebuild builds one or more targets contained in an Xcode project, or builds a scheme contained in an Xcode workspace or Xcode project.
> 
> 
> Usage
> 
> To build an Xcode project, run xcodebuild from the directory containing your project (i.e. the
     directory containing the name.xcodeproj package). If you have multiple projects in the this
     directory you will need to use -project to indicate which project should be built.  By default,
     xcodebuild builds the first target listed in the project, with the default build configuration.
     The order of the targets is a property of the project and is the same for all users of the
     project.			
To build an Xcode workspace, you must pass both the -workspace and -scheme options to define the
     build.  The parameters of the scheme will control which targets are built and how they are built,
     although you may pass other options to xcodebuild to override some parameters of the scheme.		
     There are also several options that display info about the installed version of Xcode or about projects or workspaces in the local directory, but which do not initiate an action.  These
     include -list, -showBuildSettings, -showdestinations, -showsdks, -usage, and -version.
> 

总结一下：

* 需要在包含 `name.xcodeproj` 的目录下执行 `xcodebuild` 命令，且如果该目录下有多个 projects，那么需要使用 `-project` 指定需要 build 的项目。
* 在不指定 `build` 的 `target` 的时候，默认情况下会 build project 下的`第一个 `target
* 当 build workspace 时，需要同时指定 `-workspace` 和 `-scheme` 参数，scheme 参数控制了哪些 targets 会被 build 以及以怎样的方式 build。
* 有一些诸如 `-list, -showBuildSettings, -showsdks` 的参数可以查看项目或者工程的信息，不会对 build action 造成任何影响，放心使用。

那么，xcodebuild 究竟如何使用呢？ 继续看文档:

>
>SYNOPSIS
>
>xcodebuild [-project name.xcodeproj][[-target targetname] ... | -alltargets]
 ​               [-configuration configurationname][-sdk [sdkfullpath | sdkname]] [action ...]
 ​               [buildsetting=value ...] [-userdefault=value ...]


>
>xcodebuild [-project name.xcodeproj] -scheme schemename [[-destination destinationspecifier] ...]
 ​               [-destination-timeout value] [-configuration configurationname]
 ​               [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...]
 ​               [-userdefault=value ...]
>
>xcodebuild -workspace name.xcworkspace -scheme schemename
 ​               [[-destination destinationspecifier] ...][-destination-timeout value]
 ​               [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...]
 ​               [buildsetting=value ...] [-userdefault=value ...]
>
>xcodebuild -version [-sdk [sdkfullpath | sdkname]] [infoitem]
>
>xcodebuild -showsdks
>
>xcodebuild -exportArchive -archivePath xcarchivepath -exportPath destinationpath
 ​               -exportOptionsPlist path
>
>等等
>
>

挑几个常用的形式介绍一下：

* `xcodebuild -showsdks`: 列出 Xcode 所有可用的 SDKs。
* `xcodebuild -showBuildSettings`: 查看当前工程 build setting 的配置参数。
* `xcodebuild [-project name.xcodeproj] [[-target targetname] ... | -alltargets] build`: 会 build 指定 project，其中 -target 和 -configuration 参数可以使用 xcodebuild -list 获得，-sdk 参数可由 xcodebuild -showsdks 获得。
* `xcodebuild -workspace name.xcworkspace -scheme schemename build`:build 指定 workspace，当我们使用 CocoaPods 来管理第三方库时，会生成 xcworkspace 文件，这样就会用到这种打包方式。

## 打包过程

`开始打包`---->`archive文件`---->`ipa包`

#### 生成archive文件

首先看一下生成`archive文件`的命令：

```

xcodebuild archive -workspace 项目名称.xcworkspace 
                   -scheme 项目名称 
                   -configuration 构建配置 
                   -archivePath archive包存储路径 
                    CODE_SIGN_IDENTITY=证书 
                    PROVISIONING_PROFILE=描述文件UUID
```


* workspace 这个就是项目名
* scheme 可以通过xcodebuild -list获取
* configration 一些参数，也可以通过xcodebuild -list获取，一般使用Debug、Release
* archivePath archive后的路径
* CODE_SIGN_IDENTITY 证书的Inentity
* PROVISIONING_PROFILE 描述文件UUID

如果使用Xcode的自动管理证书功能，则`CODE_SIGN_IDENTITY `和`CODE_SIGN_IDENTITY `参数不需要添加。



#### 生成ipa文件

同样看一下生成ipa文件的命令：

```

xcodebuild -exportArchive -archivePath archive文件的地址.xcarchive 
                          -exportPath 导出的文件夹地址 
                          -exportOptionsPlist exprotOptionsPlist.plist 
                          CODE_SIGN_IDENTITY=证书 
                          PROVISIONING_PROFILE=描述文件UUID

```

官方解释：

>
>Exports the archive MyMobileApp.xcarchive to the path ExportDestination using the
 ​             options specified in export.plist.
>

同样，如果你不需要的指定证书和Provisioning文件，可以把上面的两个参数去掉，它会根据你的Xcode配置去匹配。


`exportOptionsPlist`这个参数，对应一个plist文件,用来配置一些打包时需要配置的选项：

```
<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>teamID</key>
        <string>UA21BCDJHK3</string> //TeamID
        <key>method</key>
        <string>ad-hoc</string> //ad-hoc打包
        <key> compileBitcode</key> //是否编译bitcode
        <false/>
    </dict>
    </plist>

```

`exportOptionsPlist.plist`可配置的字段，可以使用`xcodebuild --help`命令查看。

至此你已经能通过命令生成ipa包了。


## 上传第三方平台

一般第三方平台都会开放上传app包的API，这里以咱们使用的`fir`平台为例：

查看`fir`的文档，可以在`文档`中看到`发布应用`的选项卡，其中有获取上传凭证的API：

>
>POST http://api.fir.im/apps
>

上传文件的API:

>
>POST upload_url
>

然后按文档格式，配置指定的参数即可。

然后利用`python`将刚才生产的ipa包，上传到fir平台即可。

## 上传到AppStore

利用`altool`: Application Loader命令行工具可以验证并上传应用程序的二进制文件到AppStore。




## 脚本化

综合以上过程，利用`python`实现脚本工具，完成一键打包并上传fir。

工具代码如下,可以在此基础代码的基础上增加更多功能：

```
# -*- coding:utf-8 -*-

import os
import sys
import requests
import time


#Release Debug
CONFIGURATION = "Debug"


def desktopPath():
    return os.path.join(os.path.expanduser("~"), 'Desktop')



#清除临时文件
def cleanArchiveFile(archivePath):
	cleanCmd = "rm -r %s" %(archivePath)
	os.system(cleanCmd)


#上传到第三方平台
def uploadIpaToPlatform(ipaPath):

    #需要的参数
    upload_url = "http://api.fir.im/apps"
    bundle_id = "com.toweringsecurities.app"
    api_token = "b95b2c2819eda5303ef0597a17f75be9"

    #获取上传凭证（上传地址）
    data = {'type': 'ios', 'bundle_id': bundle_id,
        'api_token': api_token}

    response = requests.post(url = upload_url, data = data)
    json = response.json()
    binaryDict = (json["cert"]["binary"])

    print '====' + ipaPath + '=====' + binaryDict['upload_url']

    f = open(ipaPath, 'rb')
    file_binary = {'file': f}
    param = {"key":binaryDict['key'],"token":binaryDict['token']}
    #上传ipa    
    req = requests.post(url=binaryDict['upload_url'],files=file_binary,data=param,verify=False)
    f.close()



#build过程
def xcbuild():

	#初始化
	#工程名字
	workspace = "broker-ios.xcworkspace"
	scheme = "broker-ios"

	# exportOptions.plist文件路径；根据自己的实际情况改变
	plistPath = './../../build/exportOptions.plist'

	#桌面路径
	deskPath = desktopPath()

	#archive文件导出路径
	archivePath = deskPath + '/' + scheme + '.xcarchive'

	#导出ipa文件所在文件名
	currentT = time.strftime('%Y-%m-%d-%H-%M-%S',time.localtime(time.time()))
	ipaDirName = scheme + currentT

	#ipa文件路径
	ipaPath = deskPath + '/' + ipaDirName


	# 生成archive文件的命令
	archiveCmd = "xcodebuild archive -workspace %s -scheme %s -configuration %s -archivePath %s" %(workspace,scheme,CONFIGURATION,archivePath)
	#python执行命令
	os.system(archiveCmd)
	#生成.ipa包的命令
	exportCmd = "xcodebuild -exportArchive -archivePath %s -exportOptionsPlist %s -exportPath %s" %(archivePath,plistPath,ipaPath)
	os.system(exportCmd)

	#上传fir
	uploadIpaToPlatform(ipaPath + '/' + scheme + '.ipa')

	#清除临时文件
	cleanArchiveFile(archivePath)



def main():

	#判断输入的参数
	if len(sys.argv) > 1:
		config = sys.argv[1]
		global CONFIGURATION
		if sys.argv[1]=='Release' or sys.argv[1]=='RELEASE':
			CONFIGURATION = 'Release'
		elif sys.argv[1]=='Debug' or sys.argv[1]=='DEBUG':
			CONFIGURATION = 'Debug'

	#build过程
	xcbuild()



if __name__ == '__main__':
	main()

```

为了不污染项目，该脚本和plist文件并没有放到项目工程目录中，而是把py文件设置成了全局可用，`exportOptionsPlist.plist`文件放到单独的目录，在脚本文件中引用即可。

使用的时候只需在工程目录敲命令:

如果打`Debug`包：

```
python build.py
```


如果打`Release`包:

```
python build.py Release
```


脚本文件地址在 [这里](http://git.zuinianqing.com/lantouzi/wiki/tree/master/app/topics/files/auto_packaging)