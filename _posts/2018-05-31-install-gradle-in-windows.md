---
layout: post
title: Windows에 Gradle 설치하기
subtitle: Windows에 수작업으로 Gradle 설치하기
description: Windows에 수작업으로 Gradle 설치하기
date:   2019-09-11 11:03:00 +0900
author: Jeongjin Kim
categories: Gradle
tags:	gradle
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

### 요구사항

Gradle은 Java JDK 8 이상 설치되어 있다면 주요 OS에서 대부분 설치가 가능하다.

  **자바버전 확인**
```sh
  java -version
```


Gradle은 Groovy Library를 포함하여 배포되기 때문에 특별히 따로 설치할 필요는 없다. 기존에 설치되어 있는 Groovy가 있다면 Gradle은 무시한다.

### 설치파일 다운로드

[다운로드](https://gradle.org/releases/) 페이지에서 [binary-onle](https://gradle.org/next-steps/?version=5.6.2&format=bin)나 [complete](https://gradle.org/next-steps/?version=5.6.2&format=all) 버전을 받는다.

### 압축 해제

`C:\Gradle` 디렉토리를 하나 만든다

`C:\Gradle` 디렉토리에 다운받은 압축파일을 푼다.

그러면 `C:\Gradle\gradle-5.6.2` 밑에 bin, docs, init.d 등 디렉토리가 있다.

### 환경 설정
_윈도우 키_ 를 눌러서 검색란에 _환경 변수_ 라고 입력하면 **_시스템 환경 변수 편집_** 화면 이 검색된다. 이걸 실행.

여기서 오른쪽 하단 **_환경 변수_** 클릭

**_시스템 변수_** 에 **_새로 만들기_** 클릭

변수 이름에 _GRADLE_HOME_

변수 값에 `C:\Gradle\gradle-c`

시스템 변수 중 **_Path_** 를 선택해서 **_편집_** -> **_새로 만들기_** `C:\Gradle\gradle-5.6.2\bin` 추가

### 검증
명령 프롬프트를 실행해서
```sh
gradle -v
```
이와 같은 출력이 나오면 성공
```
> 명령 프롬프트가 환경설정 전에 열려있었다면 닫고 다시 열어야한다.

------------------------------------------------------------
Gradle 5.6.2
------------------------------------------------------------

Build time:   2019-09-05 16:13:54 UTC

~~~~


```


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