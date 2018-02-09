---
title: "sslsplitの使い方"
date: 2018-02-09T15:49:47+09:00
categories: [POST, Security]
thumbnail: "images/mount22581.jpg" 
toc: true 
---

# sslsplitとは
中間者攻撃を行い、テスト対象機器が正しく証明書を検証しているか確認するツール。
sslsplitはX over SSL/TLSに幅広く使える。


## インストール
```shell
$ sudo aptitude install sslsplit
```

## 準備
証明局証明書の作成。manに書いてあるとおり。
```shell
$ cat >x509v3ca.cnf <<'EOF'
  [ req ]
  distinguished_name = reqdn

  [ reqdn ]

  [ v3_ca ]
  basicConstraints        = CA:TRUE
  subjectKeyIdentifier    = hash
  authorityKeyIdentifier  = keyid:always,issuer:always
  EOF

$ openssl genrsa -out ca.key 2048
$ openssl req -new -nodes -x509 -sha256 -out ca.crt -key ca.key \
           -config x509v3ca.cnf -extensions v3_ca \
           -subj '/O=SSLsplit Root CA/CN=SSLsplit Root CA/' \
           -set_serial 0 -days 3650
```

## HTTPとHTTPSの例
```shell
# iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-ports 10080
# iptables -t nat -A PREROUTING -p tcp --destination-port 443 -j REDIRECT --to-ports 10443

$ sslsplit -D -k ca.key -c ca.crt -l connect.log -L sample.log https 0.0.0.0 10443 http 0.0.0.0 10080
```
これは、
```shell
$ mitmproxy -e -T -p 10443
```
と同等。

## SMTPの例
```shell
# iptables -t nat -A PREROUTING -p tcp --destination-port 465 -j REDIRECT --to-ports 10465
$ sslsplit -D -k ca.key -c ca.crt -l connect.log -L sample.log ssl 0.0.0.0 10456
```
  テスト環境動作確認を行うには以下
```shell
$ openssl s_client -connect smtp.gmail.com:465 -showcerts
```


### サムネイル
[冬の蓼科山（たてしなやま）登山道](https://www.pakutaso.com/20171204352post-14526.html)
