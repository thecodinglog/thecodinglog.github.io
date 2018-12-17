---
layout: post
title: 듀얼모니터 확장, PC화면만 등 전환 단축키 만들기
description: 듀얼모니터 확장, PC화면만 등 전환 단축키 만들기
date:   2018-12-17 11:44:00 +0900
author: Jeongjin Kim
categories: windows
tags:	듀얼모니터 확장 전환 단축 windows display switch 꿀팁
---

나는 데스크탑 Windows PC 한 대와 맥북 한 대를 사용해서 작업을 한다.
모니터 하나로 일하는 것은 여러모로 불편해서 모니터 한 대를 추가 구입해 듀얼모니터로 사용하고 있다.

![](/assets/2018-12-17-shortcut-display-switch/2018-12-17-shortcut-display-switch_173414.png)
_요로케_


그런데 데스크탑을 듀얼로 쓰다가 맥북으로 소스를 전환하면 데스크탑은 여전히 듀얼 디스플레이 모드인 상태라 확장 모니터에 있던 화면들을 볼 수가 없다.

그럴때 마다 매번 디스플레이 옵션에 들어가서 화면 1에만 또는 2에만 보기를 눌러서 듀얼 모드를 껏었는데 이게 여간 귀찮은 일이 아니였다.

## 화면 전환 프로그램

윈도우에는 **displayswitch.exe** 라는 프로그램이 있는데 이걸 실행하면 윈도우 버전에 따라서 

* 화면 복사
* 확장
* PC화면만
* 두 번째 화면만

을 선택할 수 있는 창이 뜬다.

여기서 이 실행파일에 옵션을 주면 그 선택을 즉시 하는데

* `/clone`
* `/extend`
* `/internal`
* `/external`

이 각 선택지에 대한 옵션 값이다.

그래서

`%windir%\system32\DisplaySwitch.exe /internal`

을 실행하면 1번 화면에만 표시가 되고

`%windir%\system32\DisplaySwitch.exe /extend`

을 실행하면 화면 확장이되고

`%windir%\system32\DisplaySwitch.exe /external`

을 실행하면 2번 화면에만 표시가 된다.


여기서 주의할 것은 `/external` 이라고 해서 원래 모니터가 아닌 **확장 모니터라는 보장이 없다**.
확장 모니터라도 디스플레이 옵션에 1번 모니터라고 되어 있으면 이게 internal이고 2번이 external 이 된다.
다시말하면 디스플레이 옵션에 주모니터로 사용하기로한 모니터와 옵션의 internal, external은 아무 상관이 없다.


## 단축키 만들기

Windows 환경에서 커스텀 단축키를 설정하게 해주는 오토핫키 같은 매크로 프로그램들이 많다. 하지만 나는 바탕화면에 바로가기를 추가해서 이 바로가기에 단축키를 넣는 방식으로 만들어 볼 것이다.

먼저 어느 모니터가 1번인지 2번인지 확인한다.

바탕화면 우측 마우스 클릭 -> 디스플레이 설정 -> 식별 버튼 클릭


바탕화면에 바로가기를 하나 만들고 항목위치에 절대경로와 옵션을 넣고 다음
![](/assets/2018-12-17-shortcut-display-switch/2018-12-17-shortcut-display-switch_182257.png)

> `/internal` 은 1번 화면에만 표시. 만약 주화면이 2번인 경우 `/external`로 한다.

아이콘 이름을 적고 마침
![](/assets/2018-12-17-shortcut-display-switch/2018-12-17-shortcut-display-switch_182336.png)

만들어진 아이콘을 마우스 우클릭해서 속성창을 연뒤 단축키 설정하고 저장

![](/assets/2018-12-17-shortcut-display-switch/2018-12-17-shortcut-display-switch_182532.png)

이런 방법으로 단축아이콘을 하나 더 만들어서 항목 위치 입력에 옵션을 `/extend` 로 해주면 화면을 확장하는 단축키를 만들 수 있다.