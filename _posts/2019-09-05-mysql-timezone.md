---
layout: post
title: The server time zone value 'KST' is unrecognized 예외 발생시
description: The server time zone value 'KST' is unrecognized 예외 발생시
date:   2019-09-05 13:00:00 +0900
author: Jeongjin Kim
categories: Mysql IntelliJ
tags:	mysql IntelliJ
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


IntelliJ에서 로컬에서 동작하고 있는 mysql 에 직접 접속하려고 하다보니 

`The server time zone value 'KST' is unrecognized or represents more than one time zone.`

오류가 발생하였다.

개발할 때에는 Datasource의 url에 `?serverTimeZone=Asia/Seoul` 를 추가하면 대부분 해결이 된다.

IntelliJ에서는 DB 접속정보를 넣는 화면의 URL에 직접 넣으면 해결되지 않는다.

![](/assets/2019-09-05-mysql-timezone/2019-09-05-mysql-timezone_092158.png)

**Advanced** 탭에 들어가서 **serverTimezone**에 `Asia/Seoul` 를 입력하고 적용하면 해결된다.

![](/assets/2019-09-05-mysql-timezone/2019-09-05-mysql-timezone_092515.png)
