---
priority: 0.6
title: 关于非对称加密算法
excerpt: ECC实战
categories: [Summarise]
background-image: climb.jpeg
tags:
  - 算法
  - 非对称加密
  - RSA算法
  - 椭圆曲线加密算法

---

> 现在所在的公司是一家商密公司，主要涉及的应用算法就是非对称加密算法RSA和ECC。一直想能够在这方面总结一下，算是对这段时间的学习打上一个Tag。

现代密码学应用如此突飞猛进，区块链，数字货币的爆发追究源头都要感谢非对称加密算法的提出。

目前广泛应用的非对称加密算法有两种：RSA和ECC，这两种算法分别代表了两类加密算法：基于大整数因子分解 和 基于椭圆曲线的离散对数计算。

相对于RSA来说，达到相同安全等级时，ECC所需的密钥长度远低于RSA，并且ECC的秘钥对生成和计算速度也更快，更不用说ECC的存储只需要保存私钥即可，公钥可以计算得出，所以现在区块链中使用的非对称算法都是ECC算法，作为后来者的ECC大有取代RSA的架势。

两者的特性对比如下表：

|                    | RSA                 | ECC          |
| :----------------- | ------------------- | ------------ |
| 安全性             | 基于大整数因子分解  | 基于离散对数 |
| 密钥对生成速度     | 慢                  | 快           |
| 加解密速度         | 一般                | 快           |
| 同等安全性密钥位数 | 长，目前至少1024bit | 短           |
| 计算复杂度         | 亚指数级            | 完全指数级   |

我国国密局发布了一套国密算法标准，包括SM1、SM2、SM3、SM4、SSF33，其中SM1、SM4和SSF33是对称算法，SM3是摘要算法（哈希算法），SM2是非对称算法。其他的国密算法我有机会再做介绍，SM2其实是属于ECC算法的，但它选择的是椭圆曲线方程为$y^2=x^3 + ax + b$。其他的ECC数学原理是一样的，区别就在于椭圆曲线方程的选取。

先看看SM2的公私钥，SM2的公钥其实就是椭圆曲线上一个点的坐标（x, y), 那么既然曲线方程是已知的，如果知道了x，也就能计算出y，不过椭圆曲线的特性决定它同时还需要知道y的奇偶性。由此诞生了压缩公钥的定义, y如果是偶数，在x前加0x02,否则加0x03,而完整的公钥前面加0x04。

SM2的公钥可以直接由其私钥计算得出，而公钥推私钥是极难的，这样一来SM2密钥对的存储只需要存32字节的私钥。而且有压缩公钥衍生出压缩私钥的概念，这里压缩私钥并非是指私钥长度被压缩了，而是在私钥后附加一字节0x01表示它只用于生成压缩公钥。

###SM2公钥加密

用SM2的公钥加解一段信息。

![](../images/topic_resource/sm2_encrypt_sample.png)

图中显示的数据都是16进制字符串，包括原文，如果你需要加密字符串本身，往往需要确定用UTF8还是其他编码将其转为二进制数据。

这里的输出是“42C294C5D96A5C516D29797058E653F255D89B9756076642E1CDD149CCE2C47B4B8127F7A668ABEBBC8145ECA32DEF71AB2645AC4E78AB6ED16B68A31F81FEF4CE0CE649BC1D6A96DF71F7AAF716BD092BD018602D50A0D2F6BB14A0C566B4D6C8BA3477”，200个字符，也就是100个字节。

SM2公钥加密的密文是有格式的，这里密文格式是C1C3C2,其中C1是公钥，C3是Hash值，C2才是由原文加密后的密文本身。由于C1长度64字节，C3长度32字节，所以密文结果长度总是为96字节加原文长度。

有人可能要问了，C1跟上面用来加密的公钥不一样啊，C3也不是原文的SM3hash值。确实，C1和C3并非时加密公钥和原文hash值，而是加密过程中生成中间值,并且由于过程中有随机数参与，即使原文和公钥不变，加密结果也是每次都不同的。

![](../images/topic_resource/sm2_encrypt_process.png)

### SM2私钥签名

相对公钥加密来说，私钥签名多了些前置步骤。

RSA做签名是要将签名报文做Hash后，再补足到模长作为私钥加密的输入。hash算法一般使用MD5、SHA1 - 512系列，补足的话PKCS1规范里有说明RSA做加解密和签名怎么做补足。

SM2的输入同样哈希运算结果，而且因为SM3输出长度与SM2输入长度相同，所以不需要补足，但是国密标准要求SM3哈希的内容并非时签名报文本身，而是一个叫做预处理的值 Z后接报文M，即SM2的输入为SM3(Z || M)。

SM2签名结果固定64字节，同样由于随机数的参与，每次结果都是不同的。

如果要从外部导入一个SM2密钥对，需要用数字信封的方式保护私钥，国密规范GM/T 0017 - 2012还明确定义了密钥对的数字信封的格式：

>  数字信封是指用随机的对称秘钥，加密所需传输信息，然后再用对方的公钥加密对称秘钥，最后将私钥密文和对称秘钥密文一起发送非对方，这样就可以保证只有对方的私钥才能获得对称密钥并最终得到信息。

```c++
typedef struct Struct_ECCCIPHERBLOB{
	BYTE  XCoordinate[ECC_MAX_XCOORDINATE_BITS_LEN/8]; //SM2加密算法输出密文中C1的X坐标值，其位长度与设备中签名公钥的位长度相等
	BYTE  YCoordinate[ECC_MAX_XCOORDINATE_BITS_LEN/8]; //SM2加密算法输出密文中C1的Y坐标值，其位长度与设备中签名公钥的位长度相等
	BYTE  HASH[32]; //SM2加密算法输出密文中的参数C3
	ULONG CipherLen; //SM2加密算法输出密文中参数C2的长度
	BYTE  Cipher[1]; //SM2加密算法输出密文中参数C2的值
} ECCCIPHERBLOB, *PECCCIPHERBLOB;

typedef struct Struct_ECCPUBLICKEYBLOB{
	ULONG	BitLen;											//模数的实际位长度,必须是8的倍数
	BYTE	XCoordinate[ECC_MAX_XCOORDINATE_BITS_LEN/8];	//曲线上点的X座标,有限域上的整数 #define ECC_MAX_XCOORDINATE_BITS_LEN 512	
	BYTE	YCoordinate[ECC_MAX_YCOORDINATE_BITS_LEN/8];	//曲线上点的X座标,有限域上的整数 #define ECC_MAX_XCOORDINATE_BITS_LEN 512
}ECCPUBLICKEYBLOB, *PECCPUBLICKEYBLOB;

typedef struct Struct_ECCPRIVATEKEYBLOB{
	ULONG	BitLen;									//模数的实际位长度,必须是8的倍数
	BYTE	PrivateKey[ECC_MAX_MODULUS_BITS_LEN/8];	//有限域上的整数 #define ECC_MAX_MODULUS_BITS_LEN 512
}ECCPRIVATEKEYBLOB, *PECCPRIVATEKEYBLOB;

typedef struct SKF_ENVELOPEDKEYBLOB{
	ULONG				Version;					// 当前版本为 1
	ULONG				ulSymmAlgID;				// 用于加密待导入ECC密钥对的对称算法标识，限定为采用ECB模式对密钥对数据进行加密
	ULONG				ulBits;						// 待导入加密密钥对的密钥位长度
	BYTE				pbEncryptedPriKey[64];		// 待导入加密密钥对私钥的密文
	ECCPUBLICKEYBLOB	PubKey;						// 待导入加密密钥对的公钥
	ECCCIPHERBLOB		ECCCipherBlob;				// 用保护公钥加密的对称密钥密文。
}ENVELOPEDKEYBLOB, *PENVELOPEDKEYBLOB;
```

一般来说外部导入的密钥对只能用来做加解密，想做签名的话需要直接在设备内生成密钥对且私钥无法导出，原因显而易见，如果你的签名秘钥对可以导入导出，身份鉴别也就没有可信度可言。





