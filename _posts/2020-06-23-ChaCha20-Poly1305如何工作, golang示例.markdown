---
layout: post
title:  "ChaCha20-Poly1305如何工作, golang示例"
date:   2020-06-23 10:28:09 +0800
categories: jekyll update
---
使用RFC 7539中定义的ChaCha20-Poly1305算法对消息进行加密和解密。

1.1什么是ChaCha20-Poly1305 ？ 
ChaCha20-Poly1305表示使用Poly1305身份验证器以AEAD模式运行的ChaCha20 （加密和解密算法）。

1.2什么是AE或AEAD？ 

认证加密（AE）和带有关联数据的认证加密（AEAD）是一种加密消息并一起认证加密的形式。

1.3认证加密意味着什么？ 
确保没有人修改密文（加密的消息），它的工作方式类似于验证文件的SHA或MD5哈希。 
Poly1305生成一个MAC （消息验证码）（128位，16字节），并将其附加到ChaCha20密文（加密的文本）中。 
在解密过程中，该算法检查MAC以确保没有人修改密文。

1.4 ChaCha20-Poly1305如何工作？ 
ChaCha20加密使用密钥和IV（初始化值，nonce）将明文加密为等长的密文。 
Poly1305生成一个MAC（消息认证码）并将其附加到密文中。 最后，密文和明文的长度不同。

1.5我可以将同一随机数重用于不同的密钥吗？ 
不可以，每个加密的随机数和密钥都必​​须是唯一的，否则密文会妥协！

---
package main

import (
	"crypto/sha256"
	"fmt"
	"golang.org/x/crypto/chacha20poly1305"
	"os"
)

func main() {
	pass := "Hello"
	msg := "Pass"
	argCount := len(os.Args[1:])
	if argCount > 0 {
		msg = string(os.Args[1])
	}
	if argCount > 1 {
		pass = string(os.Args[2])
	}
	key := sha256.Sum256([]byte(pass))
	aead, _ := chacha20poly1305.NewX(key[:])
	if pass == "" {
		a := make([]byte, 32)
		copy(key[:32], a[:32])
		aead, _ = chacha20poly1305.NewX(a)
	}
	if msg == "" {
		a := make([]byte, 32)
		msg = string(a)

	}
	nonce := make([]byte, chacha20poly1305.NonceSizeX)
	ciphertext := aead.Seal(nil, nonce, []byte(msg), nil)
	plaintext, _ := aead.Open(nil, nonce, ciphertext, nil)
	fmt.Printf("Message:\t%s\n", msg)
	fmt.Printf("Passphrase:\t%s\n", pass)
	fmt.Printf("\nKey:\t%x\n", key)
	fmt.Printf("Nonce:\t%x\n", nonce)
	fmt.Printf("\nCipher stream:\t%x\n", ciphertext)
	fmt.Printf("Plain text:\t%s\n", plaintext)
}
---


[https://medium.com/asecuritysite-when-bob-met-alice/go-and-chacha-6645684e7d]: https://medium.com/asecuritysite-when-bob-met-alice/go-and-chacha-6645684e7d
[rfc7539]:   https://tools.ietf.org/html/rfc7539
