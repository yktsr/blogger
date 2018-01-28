---
title: "AES CBCモードで初期化ベクトルと鍵を指定して暗号化・復号"
date: 2018-01-22T15:22:10+09:00
categories: [POST, Security]
thumbnail: "images/cat21908.jpg" 
toc: true 
---

# AES CBCモードで初期化ベクトルと鍵を指定して暗号化・復号

## openssl options
```shell
options are
-in <file>     input file
-out <file>    output file
-pass <arg>    pass phrase source
-e             encrypt
-d             decrypt
-a/-base64     base64 encode/decode, depending on encryption flag
-k             passphrase is the next argument
-kfile         passphrase is the first line of the file argument
-md            the next argument is the md to use to create a key
                 from a passphrase.  One of md2, md5, sha or sha1
-S             salt in hex is the next argument
-K/-iv         key/iv in hex is the next argument

```

## 暗号化
```shell
$ openssl aes-256-cbc -e -iv 000000 -K 000000 -in file.in -out file.enc
```

## 復号
```shell
$ openssl aes-256-cbc -d -iv 000000 -K 000000 -in file.enc -out file.dec
```

## ファイルの中身を16進数文字列として得る
```shell
$ od -An -tx1 < hogehoge | tr -d ' ' | tr -d '\n'
```
