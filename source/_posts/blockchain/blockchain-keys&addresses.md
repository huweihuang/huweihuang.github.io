---
title: "[Blockchain] 区块链系列（四）之密钥和地址"
catalog: true
date: 2018-6-26 15:25:24
type: "tags"
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1542285471/header/building.jpg"
tags:
- Blockchain
catagories:
- Blockchain
---

## 1. 比特币密钥

比特币的所有权是通过`数字密钥`、`比特币地址`和`数字签名`来确定的，数字密钥不存在网络中，而由用户生成，存储在一个文件或简单的数据库中，一般称之为`钱包`。

比特币交易都需要一个有效的签名才会被存储到区块链中，只有有效的密钥才能产生有效的数字签名，因此`拥有密钥副本就拥有了该账户的比特币控制权`。

密钥是成对出现的，由`公钥`和`私钥`组成。公钥类似银行账号，私钥类似支票签名。在比特币交易环节， 收款人的公钥是通过数字指纹代表的，称之为`比特币地址`，类似支票上的收款方。一般情况，比特币地址由一个公钥生成并对应于这个公钥。

### 1.1. 公钥加密和加密货币

比特币使用`椭圆曲线算法`作为公钥加密的基础，椭圆曲线算法是一个`不可逆`的函数，即可以由A生成B，但不能由B推导出A。

在比特币系统中，用公钥加密创建一个密钥对，用于控制比特币的获取。密钥对包括一个私钥和由其衍生的唯一公钥。公钥用于接收比特币，私钥用于比特币支付时的交易签名。由私钥生产的签名可以在不泄露私钥的同时对公钥进行验证。

支付比特币时，比特币的当前所有者需要在交易中提交其公钥和签名，比特币的所有人都可以通过提交的公钥和签名验证该交易是否有效，即确认支付者在该时刻对所交易的比特币拥有所有权。

### 1.2. 私钥和公钥

比特币包含一系列密钥对，每个密钥对包含一个公钥和私钥。`私钥`是一个随机数，私钥通过椭圆曲线算法生成`公钥`，`公钥`再通过单向加密哈希函数生成`比特币地址`。比特币使用非对称加密，使得签名只能由私钥产生，且在不泄露私钥情况下所有人都可以验证该签名p。

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1529821946/article/blockchain/P-Keys.png" width="100%">

### 1.3. 私钥

私钥可以理解为一个`随机数`，`比特币地址中资金的控制取决于相应私钥的控制权`，比特币交易中，私钥用于生成支付比特币所必需的签名以证明对资金的所有权。

**从随机数生成私钥**

选择随机性来源（`熵源`），生成一个比特币私钥在本质上与“在1到$2^{256}$之间选一个数字”无异。建议使用密码学安全的伪随机数生成器（CSPRNG），并且需要有一个来自具有足够熵值的源的种子。

### 1.4.公钥

通过椭圆曲线算法可以由私钥生成公钥，该过程是`不可逆`的过程：`K（公钥）=k（私钥）*G（常数点）`。其反向运算，被称为“寻找离散对数”——已知公钥`K`来求出私钥`k`——是极其困难的。

### 1.5. 椭圆曲线算法

`椭圆曲线加密法`是一种基于离散对数问题的`非对称`（或公钥）加密法，可以用对椭圆曲线上的点进行加法或乘法运算来表达。

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1529821946/article/blockchain/elliptic-curve.png" width="50%">

比特币使用了`secp256k1`标准所定义的一条特殊的椭圆曲线和一系列数学常数。

`secp256k1`曲线由下述函数定义，该函数可产生一条椭圆曲线：

y2 = (x3 + 7)} over (F*p*)

或

y2 mod *p* = (x3 + 7) mod *p*

上述*mod p*（素数p取模）表明该曲线是在素数阶*p*的有限域内，也写作F*p*，其中p = 2256 – 232 – 29 – 28 – 27 – 26 – 24 – 1，这是一个非常大的素数。

在椭圆曲线中，点的相加等同于从该点画切线找到与曲线相交的另一点，然后映射到x轴。

### 1.6. 生成公钥

以一个随机生成的`私钥k`为起点，将其与曲线上已定义的`生成点G`相乘以获得曲线上的另一点，也就是相应的`公钥K`。生成点是`secp256k1标准`的一部分，`比特币密钥的生成点都是相同的`，所以一个`私钥k`对应唯一的一个`公钥K`，只能单向运算，即不能由`公钥K`得出`私钥k`。

```
{K = k * G}
```

示例：

```
K = 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD * G

K = (x, y)
其中，
x = F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A
y = 07CF33DA18BD734C600B96A72BBC4749D5141C90EC8AC328AE52DDFE2E505BDB
```

## 2. 比特币地址

`比特币地址`是一个由数字和字母组成的`字符串`，可以公开，由公钥生成的比特币地址一般以数字`1`开头。

例如：1J7mdg5rbQyUHENYdx39WVWK7fsLpEoXZy

在比特币交易中，`比特币地址`一般用来表示`收款方`。

比特币地址由公钥经过`单向加密哈希算法`得到，哈希算法是一种`单向函数`，接收任意长度的输入产生指纹摘要。由公钥生成比特币地址时使用的算法如下：

- Secure Hash Algorithm (SHA)

- the RACE Integrity Primitives Evaluation Message Digest (RIPEMD)，

特别是`SHA256`和`RIPEMD160`。

**示例：**

``` shell
# K是公钥，A是生成的比特币地址。
A = RIPEMD160(SHA256(K))
```

通常用户见到的比特币地址是经过`Base58Check`编码的，该编码使用了58个字符（一种Base58数字系统）和校验码，提高了可读性、避免歧义并有效防止了在地址转录和输入中产生的错误。

**由公钥生成比特币地址的过程：**

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1529821946/article/blockchain/publicKey_to_addr.png" width="50%"/>

### 2.1. Base58和Base58Check编码

#### 2.1.1. Base58

`Base58`是一种基于文本的二进制编码格式，用在比特币和其它的加密货币中。

特点：

- 数据压缩
- 易读性
- 错误诊断功能

Base58是Base64编码格式的子集，使用大小写字母和10个数字，舍弃了一些容易错读和在特定字体中容易混淆的字符。Base58不含Base64中的0（数字0）、O（大写字母o）、l（小写字母L）、I（大写字母i），以及“+”和“/”两个字符。

即`Base58就是由不包括（0，O，l，I）的大小写字母和数字组成`（26+26+10-4=58）。

**比特币的Base58字母表**

```
123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz
```

#### 2.1.2. Base58Check

`Base58Check`编码：一种Base58格式的、有版本的、经过校验的格式，可以明确的对比特币数据编码的编码格式。

`Base58Check`编码增加了错误校验码来检查数据在转录中出现的错误。校验码长4个字节，添加到需要编码的数据之后。校验码是从需要编码的数据的哈希值中得到的，所以可以用来检测并避免转录和输入中产生的错误。

**示例：**

```
 checksum = SHA256(SHA256(prefix+data))
```

结果由三部分组成：前缀、数据和校验码。

**Base58Check编码的过程**

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1529821947/article/blockchain/base58check.png" width="70%">

**Base58Check版本前缀和编码后的结果**

| 种类                        | 版本前缀 (hex) | Base58格式 |
| --------------------------- | -------------- | ---------- |
| Bitcoin Address             | 0x00           | 1          |
| Pay-to-Script-Hash Address  | 0x05           | 3          |
| Bitcoin Testnet Address     | 0x6F           | m or n     |
| Private Key WIF             | 0x80           | 5, K or L  |
| BIP38 Encrypted Private Key | 0x0142         | 6P         |
| BIP32 Extended Public Key   | 0x0488B21E     | xpub       |

### 2.2. 密钥的格式

公钥和私钥的都可以有多种`编码格式`。一个密钥被不同的格式编码后，虽然结果看起来可能不同，但是密钥所编码数字并没有改变。这些不同的编码格式主要是用来方便人们无误地使用和识别密钥。

#### 2.2.1. 私钥的格式

私钥可以以许多不同的格式表示，所有这些都对应于相同的256位的数字。

**1）私钥表示法（编码格式）**

| 种类           | 版本   | 描述                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| Hex            | None   | 64 hexadecimal digits                                        |
| WIF            | 5      | Base58Check encoding: Base58 with version prefix of 128 and 32-bit checksum |
| WIF-compressed | K or L | As above, with added suffix 0x01 before encoding             |

**2）同样的私钥，不同的格式**

以下展示了用这三种格式所生成的私钥。

| 格式           | 私钥                                                         |
| -------------- | ------------------------------------------------------------ |
| Hex            | 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD |
| WIF            | 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn          |
| WIF-compressed | KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ         |

这些表示法都是用来表示相同的数字、相同的私钥的不同方法。虽然编码后的字符串看起来不同，但不同的格式彼此之间可以很容易地相互转换。

**3）不同编码格式的转换**

1. 将Base58Check编码解码为十六进制

   ```shell
   $ sx base58check-decode 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn
   1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd 128
   ```

2. 将十六进制转换为Base58Check编码

   ```shell
   $sx base58check-encode 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd 128 
   5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn
   ```

3. 将十六进制（压缩格式密钥）转换为Base58Check编码

   ```shell
   $ sx base58check-encode 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd01 128
   KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ
   ```

#### 2.2.2. 公钥的格式

公钥也可以用不同的格式来表示，主要分为`非压缩格式`和`压缩格式`两种形式。

公钥是在椭圆曲线上的一个点，由一对坐标（x，y）组成。公钥通常表示为前缀04紧接着两个256比特的数字。其中一个256比特数字是公钥的x坐标，另一个256比特数字是y坐标。前缀`04`是用来区分`非压缩格式`公钥，`压缩格式`公钥是以`02`或者`03`开头。

公钥的组成格式：`04<x><y>`

示例：

```shell
# x,y值
x = F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A
y = 07CF33DA18BD734C600B96A72BBC4749D5141C90EC8AC328AE52DDFE2E505BDB

# 公钥K
K=04F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A07CF33DA18BD734C600B96A72BBC4749D5141C90EC8AC328AE52DDFE2E505BDB
```

**1）压缩格式公钥**

压缩格式公钥主要为了减少比特币交易的字节数，节省运行区块链数据库节点的磁盘空间。原先公钥有520比特（包括前缀，x坐标，y坐标）。公钥是椭圆曲线上（`y2mod p = (x3 + 7) mod p`）的一个点`（x,y）`，已知x坐标，可以解方程得出y，即可省去y坐标的存储，可以减少一半字节数的存储。

压缩格式公钥的前缀是`02`和`03`，因为解方程的y值有正数和负数两个值，当我们在素数p阶的有限域上使用二进制算术计算椭圆曲线的时候，y坐标可能是奇数或者偶数，分别对应前面所讲的y值的正负符号。为了区分y坐标的两种可能值，我们在生成压缩格式公钥时，如果y是偶数，则使用02作为前缀；如果y是奇数，则使用03作为前缀。

**示例：**

```shell
# 前缀03，表示y坐标是奇数
K = 03F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A
```

**公钥压缩示意图：**

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1529821947/article/blockchain/compressed-keys.png" width="50%">

**2）压缩格式私钥**

`压缩的私钥`实际上只是表示“用于生成压缩格式公钥的私钥”，而`非压缩格式私钥`用来表明“用于生成非压缩格式公钥的私钥”。当一个私钥被使用`WIF压缩格式`导出时，不但没有压缩，而且比“非压缩格式”私钥长出一个字节。

**同样的私钥，不同的格式**

| 格式           | 私钥                                                         |
| -------------- | ------------------------------------------------------------ |
| Hex            | 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD |
| WIF            | 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn          |
| Hex-compressed | 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD*01* |
| WIF-compressed | KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ         |



参考：

- [Mastering Bitcoin](https://bitcoinbook.info/)
- https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch04.asciidoc
