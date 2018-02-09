---
title: "Certificate関連のコマンドまとめ"
date: 2018-02-06T19:27:30+09:00
categories: [POST, Security]
thumbnail: "images/gunma22675.jpg" 
toc: true 
---

# 証明書に関連するあれこれに悩まされてきたので、いろいろまとめてみる
* tmp - 証明書一般
* cacert - 証明局の証明書
* cakey - 証明局の秘密鍵
* servercert - サーバ証明書
* crl - certificate revocation list 失効リスト
* cachain - ルート証明局〜中間認証局の証明書を連結したファイル

## 便利コマンドとかハマりポイント対策まとめ
1. 証明局を作らず、証明書署名要求を自分の秘密鍵で署名して、証明書と秘密鍵をぱぱっと作る
```
$ openssl req -x509 -nodes -new -keyout tmp.key -out tmp.pem -days 3600 -newkey rsa:2048 -sha512 -subj "/C=CountryName/ST=StateOrProvinceName/L=LocalityName/O=OrganaizationName/OU=OrganizationalUnit/CN=CommanName"
```

1. 証明書表示
```
$ openssl x509 -text < tmp.crt
$ openssl s_client -showcerts -connect google.com:443
```

1. pem から der へ変換
```
$ openssl x509 -inform PEM -outform DER < cert.pem > cert.der
$ openssl crl -inform PEM -outform DER < crl.pem > crl.der
```

1. CAに同じCNの証明書を発行させる
```
$ vim index.txt.attr
unique_subject = no
```

1. 証明書目的表示(server証明書はserverのフラグが立っていないとエラー)
```
$ openssl x509 -purpose < tmp.pem
```

1. CRL表示
```
$ openssl crl -text < revoked.crl
```

1. 証明書からBEGIN CERTIFICATEの部分だけを切り出す
```
$ openssl x509 -clrext < cert.pem
```

1. revoke
```
$ openssl ca -gencrl -revoke cert.pem -config openssl.conf
$ openssl ca -gencrl -out revoked.crl -config openssl.conf
```

1. ocsp responder
```
$ openssl ocsp -index index.txt -CA cacert.pem -rsigner cacert.pem -rkey private/cakey.pem -port 80
```

1. ocsp client
```
$ openssl ocsp -issuer cacert.pem -cert servercert.pem -url http://ocsp.server.com/ -CAfile cachain.pem -header HOST=ocsp.server.com
```

1. DH param
2048bit以下だと警告がでる
```
$ openssl dhparam -out dhparams.pem 2048
```

1. 複数の証明書がまとまったファイルのテキスト出力
[stackoverflow様](https://serverfault.com/questions/590870/how-to-view-all-ssl-certificates-in-a-bundle)
```
$ openssl crl2pkcs7 -nocrl -certfile CHAINED.pem | openssl pkcs7 -print_certs -text -noout
```

### サムネイル
[谷川岳景観](https://www.pakutaso.com/20171251356post-14615.html)
