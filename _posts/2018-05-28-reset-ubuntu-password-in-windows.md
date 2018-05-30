---
layout: post
title: 윈도우10에서 사용자 패스워드 리셋하기
date:   2018-05-30 17:30:00 +0900
author: Jeongjin Kim
categories: Windows Linux
tags:	Ubuntu Linux Windows Password 윈도우10 패스워드 리셋
---
Windows 10에는 _**Bash shell**_ 을 지원한다. 보통 기본 유저로 작업들을 하지만 프로그램 설치 등 root 권한이 필요한 경우가 있다. 이때 root 패스워드가 기억이 나지 않는다면 또는 다른 사용자 계정의 패스워드가 기억이 나지 않는다면 초기화 시킬 방법이 필요하다. 그 방법에 대해서 설명한다.

**1. 윈도우에서 명령 프롬프트 실행**

**2. 우분투 기본 유저를 root 변경**
```sh
ubuntu.exe config --default-user root
```
**3. 우분투 실행**
  root 계정으로 접속됨

**4. 패스워드 변경**
```sh
passwd root
```

**5. 다시 우분투 기본 유저를 기존 사용자로 변경**
```sh
ubuntu.exe  config --default-user cothe
```
