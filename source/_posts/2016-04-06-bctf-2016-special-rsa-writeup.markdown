---
layout: post
title: "BCTF 2016 Special_RSA Writeup"
date: 2016-04-06 23:42:55 +0900
comments: true
categories: ctf
published: false
---

Description of the problem is as below.

```
While studying and learning RSA, I knew a new form of encryption/decryption with the same safety as RSA.

I encrypted msg.txt and got msg.enc as an example for you.

$ python special_rsa.py enc msg.txt msg.enc
Can you recover flag.txt from flag.enc?

special_rsa.zip.f6e85b8922b0016d64b1d006529819de
```

From the description, we can guess that `msg.txt` is encrypted using algorithm similar to RSA. When you download the file and unzip it, it contains 4 files are included.

<picture>

`special_rsa.py` is the script for encrypting and decrypting messages.

`msg.txt` ,`msg.enc` is plaintext and ciphertext in order. 
