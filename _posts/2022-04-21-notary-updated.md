---
priority: 0.6
title: Xcode13 公证
excerpt: MacOSX系列
categories: [Summarise]
background-image: climb.jpeg
tags:
  - Notarize
  - 公证
  - XCode13
---

Xcode12前, 内置命令altool支持命令行上转公证, Xcode13起弃用, 改为内置命令notarytool

1. App签名现需要生产和测试分离，无法直接用Xcode开发工具上传公证，需要明确上传方式以及步骤；

    公证步骤:

   1. 首先确保选择Xcode13

      `sudo xcode-select -s /path/to/Xcode13.app`

   2. 创建App专用密码

      1. 登录https://appleid.apple.com/

      2. 选择App专用密码, 点击+根据指示创建

      3. 将App专用密码保存到公证所在的Mac电脑Keychain, 起个名字,例如NOTARI_KEY

         `xcrun notarytool store-credentials "NOTARI_KEY" --apple-id "AC_USERNAME" --team-id <WWDRTeamID> --password <secret_2FA_password>`

         其中, `AC_USERNAME`就是苹果开发账号AppId, `secret_2FA_password`即创建的App专用密码,  `WWDRTeamID`可以在开发账号中找到, 也可以通过下面的命令行获取

         `xcrun altool --list-providers -u "AC_USERNAME" -p <secret_2FA_password>`

         显示结果中的WWDRTeamID列,就是所需的teamid, 可能有多个, 是因为一个账号可能被关联到多个组.

   3. 然后调用norarytool命令,上传需要公证的应用或安装包的压缩文件, 苹果会递归的给压缩包内安装包和应用做公证

      `xcrun notarytool submit xxxxx.zip  --keychain-profile "NOTARI_KEY"  --wait  --webhook "https://example.com/notarization"`

      其中NOTARI_KEY为上一步创建的Keychain项, --wait可以等待公证完成,否则需要循环查状态, --webhook允许提供一个回调,看需要与否

   4. 上传成功后返回信息会包括一个notary-id, 可以用于公证状态的查询,也可以用以下命令查询上传历史

      `xcrun notarytool history --keychain-profile "NOTARI_KEY"`

   5. 通过以下命令,查询公证详细结果, 失败的话能查到失败的原因

      `xcrun notarytool log <notary-id> --keychain-profile "NOTARI_KEY" developer_log.json`

   6. 公证成功后, GateKeeper可以在线查询应用的公证小票,为了保证网络不可用的时候依然可以通过GateKeeper的验证,可以通过命令将公证小票装订到应用上, 注意这里不能用zip文件直接进行装订,需要具体的app,bundle, dmg或者pkg文件.

      `xcrun stapler staple "YOURAPP.app"`

      

2. App公证流程是否涉及到苹果开发者账号二次认证以及应用特殊密码，如何申请和维护；

   公证流程可以通过申请和使用App专用密钥来避免账号的双因子认证. 申请和维护方式可以参照上面公证流程第二步.

3. Pkg签名是否存在失败情况，如果签名失败如何解决，是否影响对客体验；

   签名失败或者未签名的pkg在用户点击运行时将会被系统拦截,并提示“无法打开此xxx, 因为来自身份不明的开发者”. 客户需要去系统偏好设置->安全性和隐私->通用解锁和更改,允许运行被拦截的pkg.

   签名失败往往是因为开发者证书不匹配,比如必须是有效的Developer ID证书, 也有可能是网络原因,因为公证需要每个公证的文件都带有时间戳(通过在build settings里OTHER_CODE_SIGN_FLAGS字段加上-timestamp,签名时就会带时间戳),导致签名的时候需要访问苹果的时间戳服务器(timestamp.apple.com).

   公证失败原因有多种, 常见的包括缺少有效的签名、未开启Hardened Runtime、缺少时间戳、SDK版本过低(10.9以上)等.

   无效的公证或者未公证的pkg和应用在首次启动时, 系统同样会拦截,并提示“打不开xxx,因为Apple无法检查是否包含恶意软件".不过可以右键菜单中选择打开来继续启动.

4. Pkg公证是否和App公证流程和方法一致等；

   pkg与App公证流程一致, 公证时会为所有内嵌的文件生成公证的小票.

   pkg与App的签名流程区别在于使用了不同的签名证书, pkg要用Developer ID installer类型的证书.

参考:

1. [Customizing the Notarization Workflow](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution/customizing_the_notarization_workflow/)
2. [Notarizing macOS Software Before Distribution](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution)
3. [Resolving Common Notarization Issues](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution/resolving_common_notarization_issues#3087733)