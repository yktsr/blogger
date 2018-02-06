---
title: "Hugo_command"
date: 2018-02-06T19:21:58+09:00
categories: [POST, Other]
thumbnail: "images/author.jpg" 
toc: true 
draft: true
---


## コンテンツ作成
```
$ hugo new post/first_content.md
```

## テストサーバ起動
```
$ hugo server --theme=hugo_theme_robust --buildDraft
```

## Publish
```
$ hugo undraft post/first_content.md
$ hugo --theme=hugo_theme_robust
```


