---
layout: post
title: "二、golang和gin框架安装和使用"
date: 2022-2-24
desc: golang和gin框架安装和使用
image: 
optimized_image: 
description: golang和gin框架安装和使用
category: golang ang gin

---


这篇文章主要是项目的搭建，基础的配置，结果是能运行访问接口。话不多说，让我们开始吧。

项目配置：
在conf文件夹下创建app.ini文件，这个是整个项目的配置文件conf/app.ini
```conf
#app相关
[app]
PageSize = 20                           #分页每页返回的size
JwtSecret = 1111$1111                #JWT密钥
JwtExpireTime = 100                     #jwt失效时间，单位是小时

RuntimeRootPath = runtime/              #保存文件的跟路径

ImagePrefixUrl = http://127.0.0.1:9999  #图片url
ImageSavePath = upload/images/          #图片路径
ImageMaxSize = 1                        #图片最大尺寸，单位（M）
ImageAllowExts = .jpg,.jpeg,.png        #图片允许的格式

ApkSavePath = upload/apks/              #apk文件路径
ApkAllowExt = .apk                      #apk文件格式
#iOS应用在App Store中的地址，用于版本更新
AppStoreUrl = null

LogSavePath = logs/                     #日志文件路径
LogSaveName = log                       #日志文件名称
LogFileExt = log                        #日志文件后缀
TimeFormat = 20060102                   #时间相关

WechatAppID = null       #微信的appID
WechatSecret = null #微信的secret
QQAppID =  null                           #QQ的appID
QQAppKey =  null   #QQ的appkey

#服务器相关
[server]
RunMode = debug                         #debug or release
HttpPort = 9999                         #端口号
ReadTimeout = 60                        #读取超时时间
WriteTimeout = 60                       #写入超时时间

#数据库mysql相关
[database]
Type = mysql                            #数据库类型
User = root                             #数据库账号
Password = 123456                     #数据库密码
Host = 127.0.0.1:3306                   #数据库地址+端口号
Name = api                            #数据库名称
TablePrefix = api_                    #数据库表名称前缀
```
每个配置字段的含义都有注释，解释下几个字段，JwtSecret：json web token加密的密钥，需要客户端和服务端保持一致ImagePrefixUrl：项目中返还给客户端图片的域名， AppStoreUrl：项目在AppStore中的地址，用做更新使用
其它例如数据库信息，根据自己的情况自行设置
安装ini配置包终端执行
`go get -u github.com/go-ini/ini`
配置文件已经有了，如何使用到配置文件设置的值呢？这里使用了映射结构体的方式，来获取配置文件中设置的值。
首先创建pkg/setting/setting.go文件，编写结构体，结构体与配置文件的值一一对应pkg/setting/setting.go
```go
package setting

import "time"

//app配置
type App struct {
    JwtSecret string                //jwt密钥
    JwtExpireTime time.Duration     //jwt失效时间，单位是小时
    PageSize int                    //分页返回的数据个数
    RuntimeRootPath string          //保存文件的跟路径

    ImagePrefixUrl string           //图片url
    ImageSavePath string            //图片要保存的路径
    ImageMaxSize int                //图片最大尺寸
    ImageAllowExts []string         //图片允许的格式   jpg jpeg png

    ApkSavePath string              //apk文件路径
    ApkAllowExt string              //apk文件格式
    AppStoreUrl string              //iOS应用在App Store中的地址，用于版本更新

    LogSavePath string              //日志文件保存的路径
    LogSaveName string              //日志文件名称
    LogFileExt string               //日志文件后缀
    TimeFormat string               //文件的日期名称

    WechatAppID string              //微信的appID
    WechatSecret string             //微信的secret
    QQAppID string                  //QQ的appID
    QQAppKey string                 //QQ的appkey
}
var AppSetting = &App{}

//服务配置
type Server struct {
    RunMode string              //运行模式
    HttpPort int                //端口号
    ReadTimeout time.Duration   //读取超时时间
    WriteTimeout time.Duration  //写入超时时间
}
var ServerSetting = &Server{}

//数据库配置
type Database struct {
    Type string                 //数据库类型
    User string                 //数据库用户
    Password string             //数据库密码
    Host string                 //数据库地址+端口号
    Name string                 //数据库名称
    TablePrefix string          //数据库数据表前缀
}
var DatabaseSetting = &Database{}
```
然后创建一个初始化的方法，方法的作用是把配置文件映射到上面的结构体当中，注意这个方法不能用init方法，因为要保证程序启动后首先要加载配置文件。避免多个包下init方法的执行顺序这种情况。pkg/setting/setting.go
...  //为了节省文章篇幅，以后将会用这种方式来标示已写过的代码
```go
//将配置选项映射到结构体上
func SetUp()  {
    Cfg,err := ini.Load("conf/app.ini") //加载配置文件ini
    if err != nil {
        log.Fatal("获取.ini配置失败")
    }

    //映射配置
    err = Cfg.Section("app").MapTo(AppSetting)
    if err != nil {
        log.Fatalf("Cfg配置文件映射 AppSetting 错误: %v", err)
    }

    AppSetting.ImageMaxSize = AppSetting.ImageMaxSize * 1024 * 1024     //设置允许上传图片的最大尺寸

    err = Cfg.Section("server").MapTo(ServerSetting)
    if err != nil {
        log.Fatalf("Cfg配置文件映射 ServerSetting 错误: %v", err)
    }

    ServerSetting.ReadTimeout = ServerSetting.ReadTimeout * time.Second
    ServerSetting.WriteTimeout = ServerSetting.ReadTimeout * time.Second

    err = Cfg.Section("database").MapTo(DatabaseSetting)
    if err != nil {
        log.Fatalf("Cfg配置文件映射 DatabaseSetting 错误: %v", err)
    }
}
```
接下来就是如何使用，在项目根目录下创建main.go文件，作为项目的入口，在main方法中调用SetUp方法（可以把上篇文章创建的test.go文件删除了）main.go
```go
package main

import (
    "api/pkg/setting"
    "log"
)

func main() {
    log.Println("Hello, api 正在启动中...")
    setting.SetUp() //初始化配置文件
    log.Println(setting.ServerSetting.HttpPort) //测试能否打印出ini配置文件设置的信息
}
```
然后终端执行go run main.go,看下能否正常打印出信息。以后有需要再添加的配置信息，就按照这种方法新增就可以了。
编写自定义api接口的返回码和信息
每个api接口返回的是json数据，定义的json格式如下：
```json
{
    "Code":200,
    "Msg":"Message",
    "Data":Object
}
```
Code：api接口返回的状态，200代表正常，此外还有其它状态，我们要编写的就是这个Msg：状态码所对应的消息Data：返回的数据就在这里，这个值的类型不是固定的。
pkg/e/code.go接口的状态码
```go
package e

const (
    SUCCESS = 200                               //成功响应请求
    ERROR = 500                                 //错误响应请求
    INVALID_PARAMS = 400                        //请求参数无效

    ERROR_EXIST_TAG = 10001                     //标记错误
    ERROR_NOT_EXIST_TAG = 10002                 //错误的不存在的标记
    ERROR_NOT_EXIST_ARTICLE = 10003             //错误的不存在的文章

    ERROR_AUTH_CHECK_TOKEN_FAIL = 20001         //token无效
    ERROR_AUTH_CHECK_TOKEN_TIMEOUT = 20002      //token超时
    ERROR_AUTH_TOKEN = 20003                    //taoken错误
    ERROR_AUTH = 20004                          //无效的用户
    ERROR_EXIST_AUTH = 20005                    //手机号已存在
    ERROR_AUTH_PASSWORD = 20006                 //密码错误

    ERROR_UPLOAD_SAVE_IMAGE_FAIL = 30001        // 保存图片失败
    ERROR_UPLOAD_CHECK_IMAGE_FAIL = 30002       // 检查图片失败
    ERROR_UPLOAD_CHECK_IMAGE_FORMAT = 30003     // 校验图片错误，图片格式或大小有问题
)
```
同样，每个状态吗所对应的信息也告诉客户端，返回的值在json的Msg值中pkg/e/msg.go
```go
package e

/*
   编码消息
*/
var MsgFlags = map[int]string{
    SUCCESS : "ok",
    ERROR : "fail",
    INVALID_PARAMS : "请求参数错误",
    ERROR_EXIST_TAG : "已存在该标签名称",
    ERROR_NOT_EXIST_TAG : "该标签不存在",
    ERROR_NOT_EXIST_ARTICLE : "该文章不存在",
    ERROR_AUTH_CHECK_TOKEN_FAIL : "Token鉴权失败",
    ERROR_AUTH_CHECK_TOKEN_TIMEOUT : "Token已超时",
    ERROR_AUTH_TOKEN : "Token错误",
    ERROR_AUTH : "用户不存在",
    ERROR_EXIST_AUTH : "用户已存在",
    ERROR_AUTH_PASSWORD : "密码错误",
    ERROR_UPLOAD_SAVE_IMAGE_FAIL:"保存图片失败",
    ERROR_UPLOAD_CHECK_IMAGE_FAIL:"检查图片失败",
    ERROR_UPLOAD_CHECK_IMAGE_FORMAT :"校验图片错误，图片格式或大小有问题",
}

/*
   根据传入的编码。获取对应的编码消息
*/
func GetMsg(code int)string  {
    msg,ok := MsgFlags[code]
    if ok {
        return msg
    }
    return MsgFlags[ERROR]
}
```
同样的，以后有需要新增状态，修改以上文件新增就可以了。
启动服务和应用热更新
准备工作完成一部分了，可以先把程序服务启动来。也是在main方法中进行main.go
```go
...
func main() {
    log.Println("Hello, api 正在启动中...")
    setting.SetUp() //初始化配置文件

    router := gin.Default()

    router.GET("/test", func(context *gin.Context) {
        context.JSON(e.SUCCESS,gin.H{
            "Code":e.SUCCESS,
            "Msg":e.GetMsg(e.SUCCESS),
            "Data":"返回数据成功",
        })
    })

    s := &http.Server{
        Addr:fmt.Sprintf(":%d", setting.ServerSetting.HttpPort),        //设置端口号
        Handler:router,                                         //http句柄，实质为ServeHTTP，用于处理程序响应HTTP请求
        ReadTimeout:setting.ServerSetting.ReadTimeout,          //允许读取的最大时间
        WriteTimeout:setting.ServerSetting.WriteTimeout,        //允许写入的最大时间
        MaxHeaderBytes: 1 << 20,                                //请求头的最大字节数
    }

    /*
       使用 http.Server - Shutdown() 优雅的关闭http服务
    */
    go func() {
        if err := s.ListenAndServe(); err != nil{
            log.Printf("Listen: %s\n", err)
        }
    }()

    quit := make(chan os.Signal)
    signal.Notify(quit,os.Interrupt)
    <- quit

    log.Println("Shutdown Server ...")
    ctx, cancel := context.WithTimeout(context.Background(), 5 * time.Second)
    defer cancel()
    if err := s.Shutdown(ctx); err != nil {
        log.Fatal("Server Shutdown:", err)
    }

    log.Println("程序服务关闭退出")
}
```
可以在浏览器输入http://127.0.0.1:9999/test查看数据是否正确

￼
封装返回数据方法
由于json的第一层格式是固定的，所以封装一下返回数据的方法，能更简单的调用。pkg/util/response.go
```go
package util

import (
    "api/pkg/e"
    "github.com/gin-gonic/gin"
)

/*
   数据返回信息的model，格式如下
*/
type Response struct {
    Code int            //自定义编码
    Msg string          //自定义消息
    Data interface{}    //返回的数据
}

func ResponseWithJson(code int, data interface{}, c *gin.Context) {
    c.JSON(200,&Response{
        Code:code,
        Msg:e.GetMsg(code),
        Data:data,
    })
}
```
以后会经常使用到这个方法，直接调用util.ResponseWithJson()方法即可。

