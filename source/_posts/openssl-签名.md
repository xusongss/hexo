---
title: openssl dgst
date: 2016-06-12 00:13:06
comments: true
tags:
- openssl
- linux

---
数字签名的过程是计算出数字摘要，然后使用私钥对数字摘要进行签名，而摘要是使用md5、sha512等算法计算得出
## 基础知识
### Hash 算法
Hash （哈希或散列）算法是信息技术领域非常基础也非常重要的技术。它能任意长度的二进制值（明文）映射为较短的固定长度的二进制值（Hash 值），并且不同的明文很难映射为相同的 Hash 值。


例如计算一段话“hello blockchain world, this is yeasy@github”的 MD5 hash 值为
```
$ echo "hello blockchain world, this is yeasy@github"|md5sum
89242549883a2ef85dc81b90fb606046
```
这意味着我们只要对某文件进行 MD5 Hash 计算，得到结果为 89242549883a2ef85dc81b90fb606046，这就说明文件内容极大概率上就是 “hello blockchain world, this is yeasy@github”。可见，Hash 的核心思想十分类似于基于内容的编址或命名。


注：hash 值在应用中又被称为指纹（fingerprint）、摘要（digest）。
注：MD5 是一个经典的 hash 算法，其和 SHA-1 算法都已被 证明 安全性不足应用于商业场景。


一个优秀的 hash 算法，将能实现：
* 正向快速：给定明文和 hash 算法，在有限时间和有限资源内能计算出 hash 值。
* 逆向困难：给定（若干） hash 值，在有限时间内很难（基本不可能）逆推出明文。
* 输入敏感：原始输入信息修改一点信息，产生的 hash 值看起来应该都有很大不同。
* 冲突避免：很难找到两段内容不同的明文，使得它们的 hash 值一致（发生冲突）。
* 冲突避免有时候又被称为“抗碰撞性”。如果给定一个明文前提下，无法找到碰撞的另一个明文，称为“弱抗碰撞性”；如果无法找到任意两个明文，发生碰撞，则称算法具有“强抗碰撞性”。

很多场景下，也要求对于任意长的输入内容，输出定长的 hash 结果。

### 流行的算法
目前流行的 Hash 算法包括 MD5、SHA-1 和 SHA-2。
* MD4（RFC 1320）是 MIT 的 Ronald L. Rivest 在 1990 年设计的，MD 是 Message Digest 的缩写。其输出为 128 位。MD4 已证明不够安全。
* MD5（RFC 1321）是 Rivest 于1991年对 MD4 的改进版本。它对输入仍以 512 位分组，其输出是 128 位。MD5 比 MD4 复杂，并且计算速度要慢一点，更安全一些。MD5 已被证明不具备“强抗碰撞性”。
* SHA （Secure Hash Algorithm）是一个 Hash 函数族，由 NIST（National Institute of Standards and Technology）于 1993 年发布第一个算法。目前知名的 SHA-1 在 1995 年面世，它的输出为长度 160 位的 hash 值，因此抗穷举性更好。SHA-1 设计时基于和 MD4 相同原理，并且模仿了该算法。SHA-1 已被证明不具备“强抗碰撞性”。

### 性能

一般的，Hash 算法都是算力敏感型，意味着计算资源是瓶颈，主频越高的 CPU 进行 Hash 的速度也越快。也有一些 Hash 算法不是算力敏感的，例如 scrypt，需要大量的内存资源，节点不能通过简单的增加更多 CPU 来获得 hash 性能的提升。

### 数字摘要
顾名思义，数字摘要是对数字内容进行 Hash 运算，获取唯一的摘要值来指代原始数字内容。
数字摘要是解决确保内容没被篡改过的问题（利用 Hash 函数的抗碰撞性特点）。
数字摘要是 Hash 算法最重要的一个用途。在网络上下载软件或文件时，往往同时会提供一个数字摘要值，用户下载下来原始文件可以自行进行计算，并同提供的摘要值进行比对，以确保内容没有被修改过。

## openssl dgst 应用
openssl dgst [-md5|-sha1|...] [-hex | -binary] [-out filename] [-sign filename] [-passin arg] [-verify filename] [-prverify filename] [-signature filename] [file...]

选项说明：

* file...：指定待签名的文件。

* -hex：以hex格式输出数字摘要。如果不以-hex显示，签名或验证签名时很可能乱码。

* -binary：以二进制格式输出数字摘要，或以二进制格式进行数字签名。这是默认格式。

* -out filename：指定输出文件，若不指定则输出到标准输出。

* -sign filename：使用filename中的私钥对file数字签名。

* -signature filename：指定待验证的签名文件。

* -verify filename：使用filename中的公钥验证签名。

* -prverify filename：使用filename中的私钥验证签名。

* -passin arg：传递解密密码。若验证签名时实用的公钥或私钥文件是被加密过的，则需要传递密码来解密。

支持如下几种单向加密算法，即签名时使用的hash算法。

-md4            to use the md4 message digest algorithm

-md5            to use the md5 message digest algorithm

-ripemd160      to use the ripemd160 message digest algorithm

-sha            to use the sha message digest algorithm

-sha1           to use the sha1 message digest algorithm

-sha224         to use the sha224 message digest algorithm

-sha256         to use the sha256 message digest algorithm

-sha384         to use the sha384 message digest algorithm

-sha512         to use the sha512 message digest algorithm

-whirlpool      to use the whirlpool message digest algorithm

### 生产摘要信息
1. 随机生成一段摘要信息
```SHELL
xuss@DESKTOP-C8O1AF2 ~
$ echo 123456 | openssl md5
(stdin)= f447b20a7fcbf53a5d5be013ea0b15af
```
2. 对文件生成MD5摘要信息
```SHELL
xuss@DESKTOP-C8O1AF2 ~/openssl
$ openssl dgst -md5 test.txt
MD5(test.txt)= f447b20a7fcbf53a5d5be013ea0b15af
```
3. 对文件生成sha512
```SHELL
xuss@DESKTOP-C8O1AF2 ~/openssl
$ openssl dgst -sha512 test.txt
SHA512(test.txt)= 1caced6fca2237153d65adfb0f3dbe33b9375e9eb6df17c379f80cd37deb6e6a70159c7e898576db568b871ca1c2ffd1a2cc3205f1b50be5396096335fc29c40
```

### 签名
#### 生成私钥
```SHELL
xuss@DESKTOP-C8O1AF2 ~/
$ openssl genrsa -out prvtkey.pem 1024/2038
Generating RSA private key, 1024 bit long modulus
...............++++++
.....++++++
e is 65537 (0x10001)

$ cat prvtkey.pem
-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQCiC+OtqmFhIIj1hrSHzv+HxuPZPK5Vez8ERDRRludiSzza6IVo
ySeUfSNxvUdjFBpGHxi9v0Zu0xuz48nnXsUY+3lhKaBY93ECQ9LW1EASwfio4RI+
jGiLicyjbMAvPY2mp7tkqkHf6Cb40N9NHV4TbAwFH+CpMbDcxUq8lYugCwIDAQAB
AoGAR+QEb2B+cUuw11SatQGlpgQbw53DLhNOgsMLfwL2xkngxrGPtkw/rgXSerxh
dlhNy7wyCsRYKASqbqVpRqdpwxXYLCXBLhIjLbhPAoWC+bXMtNd6fnotrH0PsE98
yLtyX3ULU0kLn7q2+lEQLIxqu9NHqaEDRMyCfRC0YHJmF5kCQQDOtRTSM3j3VCQu
kdmJkQsqdvPK1LZlOVLHpzOzYy82G/sYpsFDAH9nzr/d8H3bZFwixOhTFXRfJ2Lq
EOU1soc3AkEAyLBhvOOcBkPfxO4O8NgDnTDfnw4wUbFZ26b4NvsSLr0uXAzBTR1b
wulem1aWKX7eefJOKaH1GpoV64UPO+/vzQJAZekjPctAzXe/avJfdRJ8ldAVvB+J
WXiclnCZ7cxtv1imQG4ehGEfb1eggtSJyHu/bSj1fdjrCerKOqpfx0ygmwJAdYmQ
BJ/NroGsGdtPFtF89GA+aBpYRFA5j4Kv1wue747PCwxRXge2yWYCibnhgnYSeJto
GcwIEEd0VRb+AB2bdQJAEaemU2VPZDzA1GY3WKcz5fUpQXmfKJuQDGwuQ23Q4YSK
V0ESooiPUJBBq1PuDeIz78iNciKqa/gknrMKQ8RQYw==
-----END RSA PRIVATE KEY-----
```
#### 生成公钥
```
xuss@DESKTOP-C8O1AF2 ~/
$ openssl rsa -in prvtkey.pem -pubout -out rsa.pub.pem
writing RSA key
$ cat rsa.pub
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCiC+OtqmFhIIj1hrSHzv+HxuPZ
PK5Vez8ERDRRludiSzza6IVoySeUfSNxvUdjFBpGHxi9v0Zu0xuz48nnXsUY+3lh
KaBY93ECQ9LW1EASwfio4RI+jGiLicyjbMAvPY2mp7tkqkHf6Cb40N9NHV4TbAwF
H+CpMbDcxUq8lYugCwIDAQAB
-----END PUBLIC KEY-----
```

#### 签名并HEX输出到控制台
```
xuss@DESKTOP-C8O1AF2 ~/
$ openssl dgst -md5 -hex -sign prvtkey.pem test.txt
RSA-MD5(test.txt)= 7f6cb9deebfbee3770f58ca80dd6ae706ba26c703453ee88b907adb0e8b1f37e6de3a40b0107d0963d5e922d6c5d611598eba4410ad3ee8bcbe2016888b579956be8e31cb22154a5942a32c3d4d0d1447dd6fe19c541e53733d7a74cdad570b289001715c745313b7a33f998bbecfe9b6110be6910299c41fce216f8f7471d68
```

#### 签名保持到文件
```
xuss@DESKTOP-C8O1AF2 ~/
$ openssl dgst -md5 -sign prvtkey.pem -out test.txt.sign test.txt

xuss@DESKTOP-C8O1AF2 ~/
$ ls
prvtkey.pem  test.txt  test.txt.sign

xuss@DESKTOP-C8O1AF2 ~/
$ hexdump test.txt.sign
0000000 6c7f deb9 fbeb 37ee f570 a88c d60d 70ae
0000010 a26b 706c 5334 88ee 07b9 b0ad b1e8 7ef3
0000020 e36d 0ba4 0701 96d0 5e3d 2d92 5d6c 1561
0000030 eb98 41a4 d30a 8bee e2cb 6801 b588 9579
0000040 e86b 1ce3 21b2 a554 2a94 c332 d0d4 44d1
0000050 d67d 19fe 41c5 37e5 d733 4ca7 d5da b270
0000060 0089 1517 45c7 3b31 337a 98f9 ecbb 9bfe
0000070 1061 69be 2910 419c e2fc f816 47f7 681d
0000080
```

#### 验签
验证签名的过程实际上是对待验证文件新生成签名，然后与已有签名文件进行比对，如果比对结果相同，则验证通过。所以，在验证签名时不仅要给定待验证的签名文件，也要给定相同的算法，相同的私钥或公钥文件以及待签名文件以生成新签名信息。

私钥验签
```
xuss@DESKTOP-C8O1AF2 ~/
$ openssl dgst -md5 -prverify prvtkey.pem -signature test.txt.sign test.txt
Verified OK
```
```
xuss@DESKTOP-C8O1AF2 ~/
$ openssl dgst -md5 -verify rsa.pub.pem -signature test.txt.sign test.txt
Verified OK
```
