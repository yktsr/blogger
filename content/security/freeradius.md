---
title: "Freeradius3.0のインストール"
date: 2018-02-12T02:19:49+09:00
categories: [POST, Other]
thumbnail: "images/author.jpg" 
toc: true 
---

# Docker上にfreeradiusを建てようとした備忘録

## freeradiusの引っ越し
これまではUbuntu 16.04上にfreeradiusを立てて，Apple Timemachineを認証に使っていた．
だが，思いの外仮想マシンが重たく，サーバの数を増やす必要にも迫られたため，
宅内にあるすべてのサーバをDocker上に移すことにした．
今までEAP-TLSを使ってきたので，イメージは新しく作ることにしたのだが...

## Dockerの壁
Dockerは全くの素人なわけだが，
