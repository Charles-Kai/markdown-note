---
title: AndroidAdb
date: 2017-02-05 20:49:09
tags:
---
# Android adb调试工具--清除锁屏密码误解

@(Android)[调试工具|实用教程|adb命令]

[Android Studio ADB官方文档](https://developer.android.com/studio/command-line/adb.html)

**ADB**全称为 Android Debug Bridge, 是android 里的一个调试工具, 用这个工具可以直接操作管理android模拟器或者真实的andriod设备手机。如果你安装了**Android SDK**（或者下载adb工具包，体积小）,存放sdk的platform-tools目录下，在 命令行cmd使用需要配置路径（**android_sdk/platform-tools/adb.exe** ）到环境变量里。它可为各种设备操作提供便利，如安装和调试应用，例如查看android中的数据库，提供对 Unix shell（可用来在模拟器或连接的设备上运行各种命令）的访问。该工具作为一个客户端-服务器程序，包括三个组件：
 
- **客户端** ：该组件发送命令。客户端在开发计算机上运行。您可以通过发出 adb 命令从命令行终端调用客户端；
- **后台程序** ：该组件在设备上运行命令。后台程序在每个模拟器或设备实例上作为后台进程运行；
- **服务器** ：该组件管理客户端和后台程序之间的通信。服务器在开发计算机上作为后台进程运行。

### 关于清除解锁图案

> 用户相关的文件accounts.db（gmail账号管理），gesture.key（手势识别文件），password.key（密码文件）。
>不同品牌手机系统相关文件名也会不同,例如我的手机，华为4x文件为locksettings.db（数据库文件）。
>修理店里的师傅使用一个工具叫星海神器（高通平台强刷），功能超乎想象，几乎支持所有手机品牌（特别是苹果）
>淘宝上有卖，网上大部分加密过！小米手机叫丢失解锁神器！

#### 1. 破解条件

- **手机打开USB并连接电脑** 
- **手机被ROOT，并且ADB可以直接升级为ROOT用户** 
- **配置adb路径到环境变量或者直接在cmd命令行里面切换到adb所在路径** 
   

#### 2. 破解步骤
- **打开cmd命令行，用【adb shell】命令进入shell ** 
- **利用su命令将adb提升为root用户，如果成功，行首由$变成 #，#表示root用户** 
- **进入data/system目录 ** 
-  **ls查看当前目录 ** 
- **用ls命令查看密码文件 **  
- **用rm命令删除密码文件，若是$（不是root），则会提示"rm failed for … Permission denied"，权限不足**   

``` javascript
$ adb shell	
$ su
# cd /data/system
# ls
# rm locksettings.db
```
输入reboot或手动重启手机生效。亲测华为荣耀4X有效，删除锁屏密码后，指纹解锁自动失效，所以此方法也可以破解指纹解锁！！重新设置锁屏密码后，以前设置的指纹解锁又可以用了。

#### 3. 注意事项
		看到这儿就明白了，即便手机Root+打开USB调试，也是无法通过ADB解锁手机的。因为想要想要解锁，就得删除/data/system 下的相关文件，可删除需要由Superuser或者kingroot授予ADB shell权限，而授权需要解锁打开手机后操作Superuser程序。即解锁需要用到解锁后的手机操作，就像春晚小品《开锁》中，业主黄宏要求开锁师傅林永健开锁，林永健要求黄宏出示有效证件，可证件就在锁着的箱子里头。  
		我进行了测试后发现，在授权过一次后，下次手机用USB数据线连接电脑，再次进行解锁，即便同台电脑，也是需要再次授权的。这就说明，即便你用你的电脑经过手机授权解锁过，过后想要在忘记密码时使用ADB方式解锁，也是不可能的。  
	    我觉得这是高版本的安卓系统（eg. Android 4.2 Jelly Bean 安卓果冻豆）新有的安全特性，低版本的Android如安卓2.3.8是可以通过这种方法解锁的。因为我实际测试，我的固件版本为安卓2.3.8的三星S5570（已经Root，打开USB调试），执行命令rm gesture.key，无需授权，直接即可解锁。现在我有个问题，低版本安卓系统如 Android2.3.8的手机解锁屏幕锁定密码，是否的确必须Root，还是只要打开USB调试即可？我手头没有没Root的Android2.3手机，也懒得折腾了，就不管它了。 这样看来，高版本的安卓系统也就不存在被非手机所有者恶意解锁的BUG了。

#### 4. 题外话（今年好东西都挂了）

[收费音乐神器官网](http://itwusun.com/offer.html)，[服务器接口平台AnyListen](http://www.btjson.com/index.html)

**音乐间谍**为window PC版，**音乐助手**Android版，[Shelher微博](http://weibo.com/shelher?is_hot=1)分享出来3.3版源码[百度云 ​​​​](https://pan.baidu.com/share/init?shareid=976327165&uk=4145536040)密码gria！尴尬的是朋友云免流量也不干了！
