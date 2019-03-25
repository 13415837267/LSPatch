
## Android App破解工具Xpatch的使用方法 -- 免Root实现App加载Xposed插件
### Xpatch工具基本原理
Xpatch的原理是对Apk文件进行二次打包，重新签名，并生成一个新的apk文件。
在Apk二次打包过程中，插入加载Xposed插件的逻辑，从而实现免Root Hook任意App的Java方法。

Xpatch的Hook框架底层使用的是Lody的whale，支持的平台架构有：ARM/THUMB、ARM64，支持的andrid版本有：

 - Android 5.0.0
 - Android 5.1.1
 - Android 6.0
 - Android 6.0.1
 - Android 7.1.2
 - Android 8.1.0
 - Android 9.0.0


### 工具包下载
[点击我下载最新的Xpatch Jar包][1] (也可以在Release界面下载最新包)
### Xpatch使用方法
Xpatch项目最终生成物是一个Jar包，此Jar使用起来非常简单，只需要一行命令，一个接入xposed hook功能的新的apk就生成：
```
$ java -jar XpatchJar包路径 apk文件路径
```
例如：
```
$ java -jar ../../xpatch.jar ../../wechat.apk
```
这条命令之后，在原apk文件(wechat.apk)相同的文件夹中，会生成一个名称为`wechat-xposed-signed.apk`的apk文件，这就是重新签名之后的支持xposed插件的apk。

**PS: 由于签名与原签名不一致，因此需要先卸载掉系统上已经安装的原apk，才能安装这个Xpatch后的apk**

当然，也可以增加`-o`参数，指定新apk生成的路径：
```
$ java -jar ../../xpatch.jar ../../wechat.apk -o ../../new-wechat.apk
```

更多参数类型可以使用--help查看，或者不输入任何参数运行jar包：
```
$ java -jar ../../xpatch.jar --help(可省略)
```
这行命令之后得到结果：
```
Please choose one apk file you want to process. 
options:
 -f,--force                   force overwrite
 -h,--help                    Print this help message
 -k,--keep                    not delete the jar file that is changed by dex2jar
                               and the apk zip files
 -l,--log                     show all the debug logs
 -o,--output <out-apk-file>   output .apk file, default is $source_apk_dir/[file
                              -name]-xposed-signed.apk
```
如果嫌每次命令都要输入`java -jar xpatch.jar`太麻烦，也可以将xpatch.jar的文件目录加入到系统环境变量里，这样，每次只用输入xpatch就行，关于加入环境变量的方法，可以参考apktool文件中的做法：
[Apktool install instructions][2]

### Xposed模块的加载方法
当新apk安装到系统之后，应用启动时，默认会加载所有已安装的Xposed插件(Xposed Module)。

一般情况下，Xposed插件中都会对包名过滤，有些Xposed插件有界面，并且在界面上可以设置开关，所以默认启用所有的Xposed插件的方式，大多数情形下都可行。

但在少数情况可能会有问题，比如，同一个应用安装有多个Xposed插件（wechat插件就非常多），并且都没有独立的开关界面，这样，同时启用这些插件可能会有些问题。

为了解决此问题，当应用启动时，会查找系统中所有已安装的Xposed插件，并在文件目录下生成一个文件
`mnt/sdcard/xposed_config/modules.list`，记录这些Xposed插件应用：

```
com.john.xmessengerprivacy#X Messenger Privacy

com.blanke.mdwechat#MDWechat

com.gh0u1l5.wechatmagician#微信巫师

com.example.wx_plug_in3#畅玩微信

liubaoyua.customtext#文本自定义
```
记录的方式是：`插件app包名#插件app名称`

需要禁用某个插件，只需要在该插件包名前面增加一个`#`号即可。

比如，需要禁用`畅玩微信`和`文本自定义`插件，只需要修改该文本文件，增加两个`#`号即可：

```
com.john.xmessengerprivacy#X Messenger Privacy

com.blanke.mdwechat#MDWechat

com.gh0u1l5.wechatmagician#微信巫师

#com.example.wx_plug_in3#畅玩微信

#liubaoyua.customtext#文本自定义
```
如果需要禁用所有插件，只需在所有的包名前面增加`#`。


### Issue
Xpatch是基于apk二次打包实现的，而且使用到了dex2Jar工具，因此，也存在不少的局限性。大概有以下几点：
1. 对于使用了签名校验的应用，使用Xpatch得到的apk可能无法启动，或者无法获取到网络数据，比如优酷，趣头条等。不过，这种问题不是致命性问题，既然app启动时可以加载xposed插件，那我们可以编写一个hook获取签名的方法的xposed插件，从而使校验签名能够顺利通过。具体方法可以参考这篇文章：[**Android逆向之旅---带你爆破一款应用的签名验证问题**][3]
2. 有些app可能做了app加固，导致dex2Jar工具无法将dex文件解析为jar包，从而无法生成新的apk。这种问题暂时还无法解决。
3. Hook框架使用的是lody的[Whale框架][4]，此框架存在一些不稳定性，对少数方法的hook会导致崩溃，并且在某些机型上hook会也会崩溃。
4. Xposed Hook框架暂时不支持Dalvik虚拟机。
5. 暂时不支持Xposed插件中的资源Hook.

### 可用的Xposed模块示例

 - [畅玩微信][5]
 - [微信巫师][6]
 - [MDWechat][7]
 - [文本自定义][8]
 - ...
 - ...
 - 你自己编写的Xposed模块
 
 **PS：一般来说，只要app可以被Xpatch破解，并且运行时没有做签名校验，与其相关的Xposed模块都是可用的。**

## 站在这些巨人的肩膀上

 - [Xposed][9]
 - [whale][10]
 - [dex2jar][11]
 - [AXMLPrinter2][12]
 
## License
  [Apache 2.0][13]


  [1]: https://github.com/WindySha/Xpatch/releases/download/v1.0/xpatch-v1.0.zip
  [2]: https://ibotpeaches.github.io/Apktool/install/
  [3]: https://blog.csdn.net/jiangwei0910410003/article/details/54629728
  [4]: https://github.com/asLody/whale
  [5]: https://repo.xposed.info/module/com.example.wx_plug_in3
  [6]: https://github.com/Gh0u1L5/WechatMagician/releases
  [7]: https://github.com/Blankeer/MDWechat
  [8]: https://repo.xposed.info/module/liubaoyua.customtext
  [9]: https://github.com/rovo89/Xposed
  [10]: https://github.com/asLody/whale
  [11]: https://github.com/pxb1988/dex2jar
  [12]: https://code.google.com/archive/p/android4me/downloads
  [13]: http://www.apache.org/licenses/LICENSE-2.0.html
