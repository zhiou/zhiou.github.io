---
priority: 0.6
title: NFC浅入浅出
excerpt: 从iOS13开发NFC写Tag说起
categories: [Summarise]
background-image: climb.jpeg
tags:
  - iOS13
  - NFC
  - RFID
---

WWDC2019可以说是苹果远远被低估的一届开发者大会，本次大会放出的框架和各种改变会在几年后产生巨大的影响。但这篇博文只是想介绍下iOS13在对CoreNFC框架所做的更新以及相关的NFC背景。

### 背景

NFC可以说是从RFID演化而来的一项近场通信技术，二者的关系类似深圳和广东，NFC脱胎于RFID，却有自己的技术特点和应用场景。

[RFID(**R**adio **F**requency **ID**entification)](https://zh.wikipedia.org/wiki/射频识别)是一种通过无线电信号识别特定目标的技术，并可读写数据，包含低频到微波频段，所以传输距离也从毫米(mm)级到米(m)级不等。

RFID 一般分为标签和读取器两部分，相对于NFC来说，二者的关系不可逆。依据标签内部是否供电，分为被动式、半被动式及主动式三类。

而NFC 专注于距离在10cm内的近场通信，兼容ISO 14443标准，但也有自己的ISO 15693标准。与RFID不同，NFC是双向的，它支持主动模式、被动模式和双向模式三种模式，也就是所谓的读卡器、卡模拟和P2P功能。

苹果从iPhone6开始已经具有NFC功能，并推出了ApplePay这一移动支付手段，但并没有同时开放NFC框架给开发者使用。iOS11时代了提供CoreNFC框架开放部分功能，仅限于读取NDEF格式的NFC TAG而不允许写入，直到iOS13，苹果终于对CoreNFC框架进行扩展，支持 ISO 7816, MIFARE, ISO 15693, FeliCa TAG的读写，且不限定NDEF格式。

虽然卡模拟和P2P功能似乎还遥遥无期，但读卡器功能的开放仍然对NFC在iPhone上的应用有很大的促进作用。

###CoreNFC

英文好且网络顺畅的同学可以移步WWDC2019的 相关视频: [What's New in Core NFC](https://developer.apple.com/videos/play/tech-talks/702) 和 [Core NFC Enhanceents](https://developer.apple.com/videos/play/wwdc2019/715/)。

iOS13 不仅开放了NFC的读写标前功能，在iPhoneXS及更高版本的手机上还支持后台模式读取标前功能，这一功能允许App在未启动的情况下读取所需的标签内容，在体验上可以接近Android的水平。

现在由于iOS13还是beta版本，想提前体验Core NFC的新特性的话，需要申请iOS13 beta开发者计划，并renew apple developer plan ,否则即使具备了NFC授权([Near Field Communication Tag Reader Session Formats Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_nfc_readersession_formats))，对于iOS13也是不起作用的。

iOS13新增的支持的几种标签： ISO 7816, MIFARE, ISO 15693, FeliCa TAG。

ISO7816

不但需要NFC授权，还需要在info.plist文件提供需要支持的aid(application identifier), 键值为[com.apple.developer.nfc.readersession.iso7816.select-identifiers](https://developer.apple.com/documentation/bundleresources/information_property_list/select-identifiers)

FeliCa

需要NFC授权及system code, 键值[com.apple.developer.nfc.readersession.felica.systemcodes](https://developer.apple.com/documentation/bundleresources/information_property_list/systemcodes)

MIFARE

需要NFC授权及固定的ISO7816 aid D2760000850101



