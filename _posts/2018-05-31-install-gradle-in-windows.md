---
layout: post
title: Windows에 Gradle 설치하기
date:   2018-05-30 17:30:00 +0900
author: Jeongjin Kim
categories: Gradle
tags:	gradle
---

### 요구사항

Gradle은 Java JDK 7 이상 설치되어 있다면 주요 OS에서 대부분 설치가 가능하다.

  **자바버전 확인**
```sh
  java -version
```


Gradle은 Groovy Library를 포함하여 배포되기 때문에 특별히 따로 설치할 필요는 없다. 기존에 설치되어 있는 Groovy가 있다면 Gradle은 무시한다.

### 설치파일 다운로드

[다운로드](https://gradle.org/releases/) 페이지에서 [binary-onle](https://gradle.org/next-steps/?version=4.7&format=bin)나 [complete](https://gradle.org/next-steps/?version=4.7&format=all) 버전을 받는다.

### 압축 해제

`C:\Gradle` 디렉토리를 하나 만든다

`C:\Gradle` 디렉토리에 다운받은 압축파일을 푼다. 

그러면 `C:\Gradle\gradle-4.7` 밑에 bin, docs, init.d 등 디렉토리가 있다.

### 환경 설정
_윈도우 키_ 를 눌러서 _환경 변수_ 라고 입력하면 **_시스템 환경 변수 편집_** 화면 이 검색된다. 이걸 실행.

여기서 오른쪽 하단 **_환경 변수_** 클릭

**_시스템 변수_** 에 **_새로 만들기_** 클릭

변수 이름에 _GRADLE_HOME_

변수 값에 `C:\Gradle\gradle-4.7`

시스템 변수 중 **_Path_** 를 선택해서 **_편집_** -> **_새로 만들기_** `C:\Gradle\gradle-4.7\bin` 추가

### 검증
명령 프롬프트를 실행해서
```sh
gradle -v
```
이와 같은 출력이 나오면 성공
```
------------------------------------------------------------
Gradle 4.7
------------------------------------------------------------

Build time:   2018-04-18 09:09:12 UTC
Revision:     b9a962bf70638332300e7f810689cb2febbd4a6c

Groovy:       2.4.12
Ant:          Apache Ant(TM) version 1.9.9 compiled on February 2 2017
JVM:          1.8.0_161 (Oracle Corporation 25.161-b12)
OS:           Windows 10 10.0 amd64
```
