# AES加密算法简介及Go实现


## 什么是AES

AES（**A**dvanced **E**ncryption **S**tandard）的中文名叫**高级加密标准**，又称 **Rijndael 加密法**，是[美国联邦政府](https://zh.wikipedia.org/wiki/美国联邦政府)采用的一种[区块加密](https://zh.wikipedia.org/wiki/區塊加密)标准。这个标准用来替代原先的[DES](https://zh.wikipedia.org/wiki/DES)，已经被多方分析且广为全世界所使用。经过五年的甄选流程，高级加密标准由[美国国家标准与技术研究院](https://zh.wikipedia.org/wiki/美国国家标准与技术研究院)（NIST）于 2001 年 11 月 26 日发布于 FIPS PUB 197，并在 2002 年 5 月 26 日成为有效的标准。现在，高级加密标准已然成为[对称密钥加密](https://zh.wikipedia.org/wiki/对称密钥加密)中最流行的[算法](https://zh.wikipedia.org/wiki/演算法)之一。

该算法为[比利时](https://zh.wikipedia.org/wiki/比利时)密码学家 Joan Daemen 和 Vincent Rijmen 所设计，结合两位作者的名字，以 Rijndael 为名投稿高级加密标准的甄选流程。

以上说明来自维基百科。

## AES 特性

> 在 AES 标准规范中，分组长度只能是 128 位，也就是说，每个分组为 16 个字节（每个字节 8 位）

### 秘钥长度

- **128 位：**一般记为 `AES-128`，一字节 `8 比特位`，就是秘钥长度为 `16 字节`，分组长度为 `16 字节`，加密 `10 轮`
- **192 位：**一般记为 `AES-192`，一字节 `8 比特位`，就是秘钥长度为 `24 字节`，分组长度为 `16 字节`，加密 `12 轮`
- **256 位：**一般记为`AES-128`，一字节 `8 比特位`，就是秘钥长度为 `32 字节`，分组长度为 `16 字节`，加密 `14 轮`

### 工作模式

> 参考资料：https://github.com/openssl/openssl/tree/master/crypto/aes

- **电码本模式 ECB（Electronic Codebook Book）：**  这种模式是将整个明文分成若干段相同的小段，然后对每一小段进行加密，最后进行拼接。
- **密码分组链接模式 CBC （Cipher Block Chaining）：**这种模式是先将明文切分成若干小段，然后每一小段与初始块或者上一段的密文段进行异或运算后，再与密钥进行加密。
- **计算器模式 CTR （Counter）：**计算器模式不常见，在 CTR 模式中， 有一个自增的算子，这个算子用密钥加密之后的输出和明文异或的结果得到密文，相当于一次一密。这种加密方式简单快速，安全可靠，而且可以并行加密，但是在计算器不能维持很长的情况下，密钥只能使用一次。
- ...

### 填充模式

- **NoPadding：**数据长度不对齐时使用 0 填充，否则不填充
- **PKCS7Padding：**假设数据长度需要填充 n(n>0) 个字节才对齐，那么填充 n 个字节，每个字节都是 n ;如果数据本身就已经对齐了，则填充一块长度为块大小的数据，每个字节都是块大小
- **PKCS5Padding：**PKCS7Padding 的子集，块大小固定为 8 字节。
- ...

### AES 加密大致流程

首先会将明文以`16字节`一组，然后分成若干组。但是有时明文长度不是 16 的倍数时怎么办呢？

这时候就可以用**填充模式**来把明文填充到 16 的倍数

然后按所选择的工作模式进行加密

按秘钥长度来决定进行几轮加密

## Go 标准库

在 Go 中，官方提供了 [crypto/aes](https://pkg.go.dev/crypto/aes@go1.16.3) 标准库来给我们进行加密，官方说明是这样的：

> The AES operations in this package are not implemented using constant-time algorithms. An exception is when running on systems with enabled hardware support for AES that makes these operations constant-time. Examples include amd64 systems using AES-NI extensions and s390x systems using Message-Security-Assist extensions. On such systems, when the result of NewCipher is passed to cipher.NewGCM, the GHASH operation used by GCM is also constant-time.

大致意思就是说这个库并没有指定模式，我们用的时候可以用 `cipher` 来选择加密模式

## 什么是 CBC 模式？

**密码分组链接模式 CBC （Cipher Block Chaining）**，这种模式是先将明文切分成若干小段，然后每一小段与初始块或者上一段的密文段进行异或运算后，再与密钥进行加密。

这时候就有个问题，那第一段的明文怎么加密呢？这时候就引入了**初始化向量**（英语：initialization vector，缩写为IV）。

初始化向量是随机的，就是你可以自定义这个初始化向量，不同的初始化向量加密出来的结果也不一样。

## Go 实现

在 Go 中，我们可以用官方提供的 [crypto/aes](https://pkg.go.dev/crypto/aes@go1.16.3) 标准库来给我们进行 AES 加密，不过这个库并没有给我们指定加密模式，需要我们自己通过 [crypto/cipher](https://pkg.go.dev/crypto/cipher@go1.16.3) 来选择加密模式。

### AES CBC 模式加密

首先我们可以调用  [crypto/aes](https://pkg.go.dev/crypto/aes@go1.16.3)  的函数来返回一个密码块

```go
func NewCipher(key []byte) (cipher.Block, error)
```

NewCipher 创建并返回一个新的 cipher.Block。 key参数应为 16、24 或 32 个字节长度的 AES 密钥，以选择 AES-128，AES-192 或AES-256。

比如我们要选择 AES-128 进加密，这时候就可以这么写

```go
key := "qwertyuiopasdfgh"
block,err:=aes.NewCipher([]byte(key))
```

当然，密钥 key 是自定义的，你可以更改你喜欢的密钥。

然后比如说我们要加密 `oooooo灰灰` 这个文本内容，首先就是对这个文本内容进行**填充**，这个需要我们自己实现，我们可以定义一个 **PKCS7Padding** 的填充方法，用 **PKCS7Padding** 填充模式进行填充内容。

> **PKCS7Padding：**假设数据长度需要填充 n(n>0) 个字节才对齐，那么填充 n 个字节，每个字节都是 n 。如果数据本身就已经对齐了，则填充一块长度为块大小的数据，每个字节都是块大小

```go
/*
	PKCS7Padding 填充模式
	text：明文内容
	blockSize：分组块大小
*/
func PKCS7Padding(text []byte,blockSize int) []byte {
	// 计算待填充的长度
	padding := blockSize - len(text)%blockSize
	var paddingText []byte
	if padding == 0 {
		// 已对齐，填充一整块数据，每个数据为 blockSize
		paddingText = bytes.Repeat([]byte{byte(blockSize)},blockSize)
	}else {
		// 未对齐 填充 padding 个数据，每个数据为 padding
		paddingText = bytes.Repeat([]byte{byte(padding)},padding)
	}
	return append(text,paddingText...)
}
```

填充完后，我们就可以用 [crypto/cipher](https://pkg.go.dev/crypto/cipher@go1.16.3) 的 `NewCBCEncrypter` 函数来得到一个 `BlockMode` ，然后用 `BlockMode` 来进行加密。具体代码如下：

```go
/*
	CBC 加密
	text 待加密的明文
    key 秘钥
*/
func CBCEncrypter(text []byte,key []byte,iv []byte) []byte{
	block,err:=aes.NewCipher(key)
	if err !=nil {
		fmt.Println(err)
	}
	// 填充
	paddText := PKCS7Padding(text,block.BlockSize())

	blockMode := cipher.NewCBCEncrypter(block,iv)

	// 加密
	result := make([]byte,len(paddText))
	blockMode.CryptBlocks(result,paddText)
    // 返回密文
	return result
}
```

### AES CBC 模式解密

解密的话 Go 官方标准库 [crypto/cipher](https://pkg.go.dev/crypto/cipher@go1.16.3) 为我们提供了 `NewCBCDecrypter` 函数，我们可以通过此函数来获得  `BlockMode`  ，然后进行解密

```go
/*
	CBC 解密
	encrypter 待解密的密文
	key 秘钥
*/
func CBCDecrypter(encrypter []byte,key []byte,iv []byte) []byte {
	block,err:=aes.NewCipher(key)
	if err !=nil {
		fmt.Println(err)
	}
	blockMode := cipher.NewCBCDecrypter(block,iv)
	result := make([]byte,len(encrypter))
	blockMode.CryptBlocks(result,encrypter)
	// 去除填充
	result = UnPKCS7Padding(result)
	return result
}

```

上面去除填充的 `UnPKCS7Padding` 函数，我们可以通过在末尾填充的数据来获取到底填充了多少长度的数据。因为 **PKCS7Padding** 填充数据的原理是假设数据长度需要填充 n(n>0) 个字节才对齐，那么填充 n 个字节，每个字节都是 n 。如果数据本身就已经对齐了，则填充一块长度为块大小的数据，每个字节都是块大小

```go
/*
	去除 PKCS7Padding 填充的数据
	text 待去除填充数据的原文
*/
func UnPKCS7Padding(text []byte) []byte{
	// 取出填充的数据 以此来获得填充数据长度
	unPadding := int(text[len(text)-1])
	return text[:(len(text)-unPadding)]
}
```

### 完整代码


```go
package main

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"fmt"
)

func main() {
	text := "oooooo灰灰"
	key := "qwertyuiopasdfgh"
	// 初始向量
	iv:=[]byte("qwertyuiopasdfgh")

	// 加密
	result := CBCEncrypter([]byte(text),[]byte(key),iv)

	// 解密
	fmt.Println(string(CBCDecrypter(result,[]byte(key),iv)))
}
/*
	CBC 加密
	text 待加密的明文
    key 秘钥
*/
func CBCEncrypter(text []byte,key []byte,iv []byte) []byte{
	block,err:=aes.NewCipher(key)
	if err !=nil {
		fmt.Println(err)
	}
	// 填充
	paddText := PKCS7Padding(text,block.BlockSize())

	blockMode := cipher.NewCBCEncrypter(block,iv)

	// 加密
	result := make([]byte,len(paddText))
	blockMode.CryptBlocks(result,paddText)
	// 返回密文
	return result
}

/*
	CBC 解密
	encrypter 待解密的密文
	key 秘钥
*/
func CBCDecrypter(encrypter []byte,key []byte,iv []byte) []byte {
	block,err:=aes.NewCipher(key)
	if err !=nil {
		fmt.Println(err)
	}
	blockMode := cipher.NewCBCDecrypter(block,iv)
	result := make([]byte,len(encrypter))
	blockMode.CryptBlocks(result,encrypter)
	// 去除填充
	result = UnPKCS7Padding(result)
	return result
}

/*
	PKCS7Padding 填充模式
	text：明文内容
	blockSize：分组块大小
*/
func PKCS7Padding(text []byte,blockSize int) []byte {
	// 计算待填充的长度
	padding := blockSize - len(text)%blockSize
	var paddingText []byte
	if padding == 0 {
		// 已对齐，填充一整块数据，每个数据为 blockSize
		paddingText = bytes.Repeat([]byte{byte(blockSize)},blockSize)
	}else {
		// 未对齐 填充 padding 个数据，每个数据为 padding
		paddingText = bytes.Repeat([]byte{byte(padding)},padding)
	}
	return append(text,paddingText...)
}

/*
	去除 PKCS7Padding 填充的数据
	text 待去除填充数据的原文
*/
func UnPKCS7Padding(text []byte) []byte{
	// 取出填充的数据 以此来获得填充数据长度
	unPadding := int(text[len(text)-1])
	return text[:(len(text)-unPadding)]
}

```


## 总结

尽管 CBC 模式是 ECB 的改良版，但是它还是有一个 bug，问题出在初始变量 `iv` 上，这个下次在说。