---
title: "(*(void(*)()) shellcode)()の意味について考える"
date: 2018-01-16T16:28:12+09:00
categories: [POST, Security]
thumbnail: "images/void.png" 
toc: true 
---

# `(*(void(*)()) shellcode)()`ってなんぞ？
最近、アセンブリ言語の勉強を始めたわけだが、文字列を関数として呼び出すコードを書く際、決まってこの表現が出てくる。

~~~
char shellcode[] = "\x00\x00\x00\x00\x00";
(*(void(*)()) shellcode)()
~~~

なんとなく関数ポインタを指していることは予想がつくが、そもそも文字列を関数として処理させるには
どうしたらいんだろう？そんな疑問に大正義StackOverflowが答えてくれた。

[What does void mean in code](https://stackoverflow.com/questions/18448204/what-does-void-mean-in-code)

## そもそも関数ポインタってなんだっけ
~~~
#include <stdio.h>

void func(void);

int main() {
  void (*hoge)() = func;
  hoge();
  return 0;
}

void func() {
  printf("hoge hoge");
}
~~~
変数hogeは関数ポインタ（アドレス）を指す。宣言方法は**戻り値の型 (*変数名) (仮引数)**;
つまり引数がある場合はこうする。
~~~
#include <stdio.h>

int func(int, int);

int main() {
  int (*hoge)(int, int) = func;
  hoge(1,2);
  return 0;
}

int func(int a, int b) {
  return a+b;
}
~~~

## 少し書き下してみる
つまり下記を少し書き下すと、
~~~
(*(void(*)()) shellcode)()
~~~

~~~
char shellcode[] = "\x00\x00\x00\x00\x00";
void (*funcPointer)() = (void(*)())shellcode;
funcPointer();
~~~
2行目で文字を関数ポインタへキャストし、3行目で実行する。
ここから変数名と仮引数を省略し、関数呼び出しを1行で行うと上記の表現になる。

## (*(の*っている？
するとこれって
~~~
((void(*)()) shellcode)()
~~~
でもよいのでは？

この疑問にもStackOverflowは答えてくれていた！

> `(*(plainsig_t*)shellcode) ();`

>For function pointers, you don't need to dereference them, so it is shorter to just code:

>`((plainsig_t*) shellcode) ();`

やS神
