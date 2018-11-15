---
title: "[Ethereum] 以太坊系列（二）之账户管理"
catalog: true
date: 2018-8-11 18:20:24
type: "tags"
subtitle:
header-img: "https://res.cloudinary.com/dqxtn0ick/image/upload/v1542285471/header/building.jpg"
tags:
- Blockchain
- Ethereum
catagories:
- Blockchain
- Ethereum
---

## 1. 账户

以太坊有两种账户类型：

- 外部账户（EOA）
- 合约账户

所有账户的状态代表以太坊网络的状态，以太坊网络会和每一个区块一起更新，网络需要达成关于以太坊的共识。账户代表外部代理人的身份，账户运用非对称加密的私钥来签署交易，以便以太坊虚拟机可以安全验证交易发送者的身份。

## 2. 钥匙文件(Keyfiles)

每个账户都由一对密钥来定义，包括公钥和私钥。账户以地址为索引，地址由公钥生成，每对私钥/地址都编码在一个钥匙文件（JSON文件）中，其中最重要的为账户私钥，通常用创建账户时的密码加密。可以在以太坊节点数据目录的`keystore`子目录下找到钥匙文件。

## 3. 创建账户

为了从账户发送交易包括以太币，需要拥有`Keyfiles`和密码。需要记住并备份钥匙文件，没有“找回密码”的选项。

### 3.1. 使用geth account new

`geth`是以太坊的go语言客户端。创建账户只需要执行`geth account new`命令。

#### 3.1.1. 下载安装

```shell
# 下载源码
git clone https://github.com/ethereum/go-ethereum.git
# 编译
cd go-ethereum
make geth
# 编译后会在`go-ethereum/build/bin/`目录下生成二进制文件，拷贝到$PATH路径下
cp build/bin/geth /usr/local/bin/
# 验证
geth --help
```

> go的版本最好在`1.9.2`及以上版本，否则编译可能报以下错误
>
> ```shell
> ...
> # github.com/ethereum/go-ethereum/vendor/gopkg.in/olebedev/go-duktape.v3
> vendor/gopkg.in/olebedev/go-duktape.v3/duktape.go:133: constant 18446744073709551615 overflows int64
> util.go:45: exit status 2
> exit status 1
> make: *** [all] Error 1
> ```

#### 3.1.2. 创建账户

`geth account new`

```shell
# 交互式，输入密码
$ geth account new
INFO [07-09|05:06:46.393] Maximum peer count                       ETH=25 LES=0 total=25
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {df90109c1497bc6f3a0cd98cdc114ec89481447e}

# 非交互式，从文件读取密码。不建议使用，容易暴露密码
$ geth --password /path/to/password account new
# 例如
$ geth --password ethpasswd account new
INFO [07-09|05:27:48.567] Maximum peer count                       ETH=25 LES=0 total=25
Address: {0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7}
```

#### 3.1.3. 查询账户

`geth account list`

```shell
$ geth account list
INFO [07-09|05:29:31.708] Maximum peer count                       ETH=25 LES=0 total=25
Account #0: {df90109c1497bc6f3a0cd98cdc114ec89481447e} keystore:///root/.ethereum/keystore/UTC--2018-07-08T21-07-05.374654217Z--df90109c1497bc6f3a0cd98cdc114ec89481447e
Account #1: {0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7} keystore:///root/.ethereum/keystore/UTC--2018-07-08T21-27-48.567366972Z--0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7
```

`keyfiles`存储在`~/.ethereum/keystore`路径下，一般按照创建时间顺序排列，格式如下：

```shell
$ cd .ethereum/keystore
$ ls
UTC--2018-07-08T21-07-05.374654217Z--df90109c1497bc6f3a0cd98cdc114ec89481447e
UTC--2018-07-08T21-27-48.567366972Z--0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7
```

keyfile的文件内容如下：

```shell
$ cat UTC--2018-07-08T21-07-05.374654217Z--df90109c1497bc6f3a0cd98cdc114ec89481447e | python -m json.tool
{
    "address": "df90109c1497bc6f3a0cd98cdc114ec89481447e",
    "crypto": {
        "cipher": "aes-128-ctr",
        "cipherparams": {
            "iv": "ac945759e1e445776387eeb855df208d"
        },
        "ciphertext": "4c75c2dcbba248e9f0f43bb4fed4ca9be10b00c02bb60ad7926f013541266592",
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 262144,
            "p": 1,
            "r": 8,
            "salt": "199c4c1129a594871e47f3be3b71d57b8e0efcd5f543b9684495f007eb51ce1c"
        },
        "mac": "da461ede939be6509db7bf25f0b30b557d66e5edc26b9590dfbba733e4af5c7b"
    },
    "id": "7cda7e39-1132-4b0c-aade-2aed1e845ed1",
    "version": 3
}
```

### 3.2. 使用geth控制台

使用geth控制台创建新账户，需要在控制台模式下开启geth交互模式。

```shell
$ geth console 2>> file_to_log_output
Welcome to the Geth JavaScript console!

instance: Geth/v1.8.13-unstable-3b074515/linux-amd64/go1.9.4
coinbase: 0xdf90109c1497bc6f3a0cd98cdc114ec89481447e
at block: 0 (Thu, 01 Jan 1970 08:00:00 CST)
 datadir: /root/.ethereum
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

>
```

#### 3.2.1. 创建账户

`personal.newAccount()`

```shell
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0x06341612bc695a2cab1d294630ce44f8789efabc"
```

#### 3.2.2. 列出账户

`eth.accounts`

```shell
> eth.accounts
["0xdf90109c1497bc6f3a0cd98cdc114ec89481447e", "0x0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7", "0x06341612bc695a2cab1d294630ce44f8789efabc"]
```

## 4. 更新、备份、恢复账户

### 4.1. 更新账户

`geth account update`更改账户密码，需要输入原密码。

以下通过`账户地址`来修改密码：

```shell
$ geth account update df90109c1497bc6f3a0cd98cdc114ec89481447e
INFO [07-09|06:02:53.411] Maximum peer count                       ETH=25 LES=0 total=25
Unlocking account df90109c1497bc6f3a0cd98cdc114ec89481447e | Attempt 1/3
Passphrase:
INFO [07-09|06:02:57.475] Unlocked account                         address=0xdF90109c1497bc6f3a0cd98cdC114Ec89481447E
Please give a new password. Do not forget this password.
Passphrase:
Repeat passphrase:
```

也可以通过`账户索引`来修改密码：

```shell
# 以下示例的账号索引为 0
$ geth account update 0
INFO [07-09|06:05:59.087] Maximum peer count                       ETH=25 LES=0 total=25
WARN [07-09|06:05:59.087] -------------------------------------------------------------------
WARN [07-09|06:05:59.087] Referring to accounts by order in the keystore folder is dangerous!
WARN [07-09|06:05:59.087] This functionality is deprecated and will be removed in the future!
WARN [07-09|06:05:59.087] Please use explicit addresses! (can search via `geth account list`)
WARN [07-09|06:05:59.087] -------------------------------------------------------------------
Unlocking account 0 | Attempt 1/3
Passphrase:
INFO [07-09|06:06:04.081] Unlocked account                         address=0xdF90109c1497bc6f3a0cd98cdC114Ec89481447E
Please give a new password. Do not forget this password.
Passphrase:
Repeat passphrase:
```

可以通过`--password`参数修改，仍需要输入文件中的密码：

```shell
geth --password ethpasswd account update 0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7
INFO [07-09|06:10:16.564] Maximum peer count                       ETH=25 LES=0 total=25
Unlocking account 0ad31ccaf888cf8499de4cdfa18f8d746a1d27e7 | Attempt 1/3
Passphrase:
INFO [07-09|06:11:04.954] Unlocked account                         address=0x0AD31CcAf888Cf8499de4cdFA18F8d746A1d27E7
Please give a new password. Do not forget this password.
Passphrase:
Repeat passphrase:
```

### 4.2. 账户备份和恢复

**手动备份/恢复**

要从账户发送交易需要`keyfile`，Linux系统中，keyfile位于`~/.ethereum/keystore`中。

- 备份keyfile：复制keystore目录下的一个或多个keyfile。
- 恢复keyfile：将备份的keyfile重新拷贝到keystore的目录下。

**导入未加密的私钥**

```shell
geth account import /path/to/<keyfile>
```



参考：

- http://www.ethdocs.org/en/latest/account-management.html

