---
layout: post
title: git project를 github로 공개하기
subtitle: github에 처음으로 올리기
description: github에 처음으로 올리기
date:   2017-08-04 08:43:59
author: Jeongjin Kim
categories: git
tags:	github git
---
#### github에 repository 만들기
- https://github.com/[github 계정명]/[repository 이름].git
- git@github.com:[github 계정명]/[repository 이름].git

#### project를 git project로 만들기
```bash
git init
```

#### remote 등록(SSH로 접속)
```bash
git remote add [remote명] [리모트주소]
git remote add origin git@github.com:[github 계정명]/[repository 이름].git
```

#### remote tracking
```bash
git branch -u [remote명]/[remote branch명] [local branch명]
git branch -u origin/master master
```

#### local을 remote로
커밋된 자원을 push
```sh
git push
```

>push 할 때
_fatal: refusing to merge unrelated histories_
오류가 발생할 수 있다. 원인은 다양하겠지만 한 가지 예를들면 local에서 git project를 만들고 git hub에서 파일을 하나 생성해서 commit을 해놓으면 두 git project가 공통 시작점이 없기 때문에 발생한다.
이런 경우
```sh
git pull origin branchname --allow-unrelated-histories*
```
>pull 명령 뒤에 옵션을 주어 해결 할 수 있다.

---

#### https로 push 할 때 403 오류가 났을 때 시도 해볼만한 것 (26 Feb 2019, updated)
```sh
git remote set-url origin https://[github 계정명]@github.com/[github 계정명]/[repository 이름].git
```