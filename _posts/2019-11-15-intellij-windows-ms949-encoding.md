---
layout: post
title: IntelliJ에서 MS949 Encoding 문제 발생시 
description: IntelliJ에서 MS949 Encoding 문제 발생시
date:   2019-11-15 13:00:00 +0900
author: Jeongjin Kim
categories: IDE
tags:	intellij ms949 utf8 encoding
---

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>

윈도우에 설치한 IntelliJ에서 Build 할 때 한글로 된 코멘트 등으로 인해 오류가 발생하면 인코딩 설정을 적절히 변경하면 된다.

문제는 분명히 파일의 인코딩 타입은 utf-8 이고 IntelliJ의 File Encodings 옵션에 UTF-8로 설정했음에도 해결이 안되는 경우가 있다.

이럴때 VM 옵션에 인코딩 타입을 넣어서 시도해보자.

VM 옵션 메뉴로 들어가서
```
Help -> Edit Custom VM Options...
```

파일 인코딩 옵션 한 줄을 추가한다.
```
-Dfile.encoding=UTF-8
```


해결이 됐다면 럭키!