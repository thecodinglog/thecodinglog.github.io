---
layout: post
title: gradle upgrade 하기
description: gradle upgrade 하기
date:   2018-11-20 11:44:00 +0900
author: Jeongjin Kim
categories: Gradle
tags:	gradle upgrade update
---

SpringBoot를 2.1.0 으로 올리려니까 gradle plugin 버전이 4.4 이어야 한다고 오류가 떴다.

![](/assets/2018-11-20-gradle-upgrade/2018-11-20-gradle-upgrade_113112.png)

gradle을 upgrade 하자.

먼저 시스템에 설치되어 있는 gradle을 upgrade 할 것인데 Homebrew 로 설치했어서 간단하게 upgrade 가 가능했다. Homebrew는 참 좋은 것 같다.

> Mac OS, Homebrew

```sh
brew upgrade gradle
```

그러면 쭉쭉 알아서 업그레이드가 되고 (시간이 좀 걸렸다) 현재 설치 버전을 확인한다.

![](/assets/2018-11-20-gradle-upgrade/2018-11-20-gradle-upgrade_113849.png)

다음은 프로젝트 디렉토리 안에 있는 gradle wrapper의 버전을 upgrade 해보자.

먼저 현재 버전확인

![](/assets/2018-11-20-gradle-upgrade/2018-11-20-gradle-upgrade_114015.png)

warpper 가 있는 디렉토리로 가서 아래 명령어를 입력한다.

```sh
./gradlew wrapper --gradle-version=4.10.2 --distribution-type=bin
```

다시 버전을 확인해본다.

```sh
./gradlew -v
```

![](/assets/2018-11-20-gradle-upgrade/2018-11-20-gradle-upgrade_114304.png)
