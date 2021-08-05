---
priority: 0.6
title: iOS逆向
excerpt: 越狱、砸壳和重签名
categories: [Summarise]
background-image: climb.jpeg
tags:
  - 越狱
  - 砸壳
  - 重签名

---

这周因公司项目，需要对Appstore上的某App进行重签名，于是做了一番探索。

## 越狱

首先必备越狱后的iPhone一台，找到一台公司的iPhone5S测试机，系统版本10.1.1，不新不旧，正好用来越狱。

在网上找了一些越狱工具，经过多次实践，windows系统版本7.0爱思助手一次性越狱成功。

其中需要配合在手机上运行yulu102，并等待设备重启，重启后iPhone如果出现cydia 应用并能启动，说明越狱成功。

注意目前iPhone在重启后需要重新越狱。

## 脱壳

Appstore上的应用除了有开发者证书签名，还被苹果公司加过一次密，所以是直接重签名虽然能安装但是是无法运行的。

所以第二步就是脱壳。

网上很多脱壳工具，Clutch、dumpdecrypted等，这些方式都使用过，而且都有效，但问题是一次只能脱壳一个文件，而且文件拷入拷出都需要自己通过scp命令或工具进行，对于App内依赖了多个Framework的情况，就会很麻烦。

还好找到另一个工具，可以直接将整个App脱壳并且自动将其打包成ipa并拷出iPhone，这就是frida-ios-dump。

frida-ios-dump依赖一些工具，MacOSX下安装步骤如下：

```shell
brew install wget
wget https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
rm ~/get-pip.py
brew install usbmuxd
```

大概说明下上面的指令：

由于frida-ios-dump 需要用pip安装，而pip又需要wget，所以首先需要wget

brew是Mac自带的工具，但是往往先需要执行brew update更新

usbmuxd 是访问手机的重要工具，它的作用是在USB上实现多路tcp连接，我在局域网上用SSH直接访问手机的ip没能成功，最后就是靠这个工具，数据线连上手机后，执行iproxy 2222 22命令， 才能用ssh -P 2222 root@localhost 连上手机的。

然后下载 [frida](https://github.com/AloneMonkey/frida-ios-dump)

并在目录内执行下面的指令，安装frida-ios-dump：

```shell
sudo pip install six --upgrade --ignore-installed six
sudo pip install -r requirements.txt --upgrade
```



frida-ios-dump 安装成功后，仍然保持在当前目录，先执行iproxy 2222 22 指令，意思是把主机的2222端口映射到iPhone的22端口；

11月8日，同事踩坑补充：

越狱手机需要安装frida
越狱手机的安装步骤如下：
1. 启动 Cydia
2. 添加软件源 软件源 Sources-> 编辑 Edit（左上角）-> 添加 Add（右上角）-> 输入 https://build.frida.re/
3. 通过刚才添加的软件源安装 frida 插件。根据手机进行安装：iPhone 5 及之前的机器为 32 位，5s 及之后的机器为 64 位，进入 变更->找到Frida->进入Frida 在右上角点击安装

最后直接执行指令：

python dump.py XXXX(XXXX为你需要脱壳的app的名字）

这里python版本为2.7，MacOSX系统自带就是这个版本，3.x版本不保证成功。

最后可以在当前目录下看到导出的已经脱壳的ipa，这时已经可以用class-dump工具导出app内的头文件了，命令为class-dump -H xxx.ipa -o ./headers/, 当然这步可选。

还需要注意的是，在什么机器上脱壳，得到的app就是什么架构的，比如在iPhone5S上脱壳，就只能得到arm64架构的app,armv7的架构只能iPhone5以下的机器上脱壳才能得到，

所以想要获得所有架构的App，需要单独获得各个二进制文件，用lipo命令合并成FAT文件才行。

## 重签名

重签名工具也有不少，脚本和软件都有，我最后找到两个都可用

脚本： [iOS_resign_scripts](https://github.com/chenhengjie123/iOS_resign_scripts)

App:    [iReSign](https://github.com/maciekish/iReSign)

这两个都有使用说明，我就不再重复，但有几点需要注意：

embbed.mobileprovision 文件，这个在以前发布的企业证书ipa包中都有，拷贝一个就可以，其实里面就是你的开发者证书。

打包之前，先把脱壳的ipa解压，看下里面的App内的info.plist文件是否有SupportedDevices字段，有的话将其删掉，否则不在这个列表中的设备将无法安装；然后重新制作成ipa（把App放到Payload文件夹，压缩Payload文件夹为zip格式，改zip后缀为ipa即可）

最后利用工具将ipa重新签名,以iReSign为例，其中ID可以不修改。需要注意的是，embedded.mobileprovision文件和签名的证书文件需要匹配，也就是说，前者必须是后者签名的ipa包中获得。

## 问题和解决

1.  python dump.py xxxx 报错 unexpected error while probing dyld of target process

- 双击Home键，将后台运行的程序关闭后重试

2.  python dump.py xxxx 报错 Error reading SSH protocol banner[Errno 54] Connection reset by peer

- Cydia安装OpenSSH

3. iPhoneXR上没图标，甚至出现过桌面上都没有App，但搜索却能搜到且能打开的情况

问题原因：

- 没能完全确定原因，可能是由于破解后的App是经过Appstore瘦身的，有些图片资源被简化掉了，但对比其他安装包，此App内文件并没少；

解决方法：

- 将App内的Assets.car解压,非zip格式，可以通过[iOSImagesExtractorApp](https://github.com/devcxm/iOS-Images-Extractor/releases)工具解压，在解压过得文件夹里增加AppIcon@3x（Payload目录下都会有）的图片。

- 建立一个XCode工程，图片添加到Assets.xcassets；其中AppIcon图片要按照size添加到Assets.xcassets的AppIcon。

- 打包这个临时App，从中获取Assets.car 替换App里的相应压缩包。
- 打包签名App
