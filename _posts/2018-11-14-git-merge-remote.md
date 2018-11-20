---
layout: post
title: git repo가 아닌 디렉토리를 remote repo와 병합하면서 git repo로 만들기
description: git repo가 아닌 디렉토리를 remote repo와 병합하면서 git repo로 만들기
date:   2018-11-14 16:30:00 +0900
author: Jeongjin Kim
categories: git
tags:	git remote repository 병합
---

리모트에 있는 git 서버에 repogitory가 있는데 이것과 git 리포지토리가 아닌 디렉토리를 병합해야 하는 경우에 어떻게 하는지 알아본다.

예를들면 이런 경우이다.

A가 프로젝트를 만들고 틀을 짠뒤에 전체 소스를 하드카피를 해서 B에게 전달한다.
서로 각자 개발을 진행하다가 형상 관리가 필요해서 A가 본인 소스를 git repository로 만들고 remote에도 bare repository를 만들어서 upstream으로 사용한다.
B가 가지고 있는 소스도 서로 공유를 위해 A가 올린 원격 Repository와 연결을 하고자 할 때이다.


## 테스트 상황 만들기

### 원격지 bare repository 만들기
```sh
$ mkdir gittest
$ cd gittest
gittest$ git init --bare --shared base.git
```

### 샘플 데이터 만들고 원격지에 올리기
```sh
gittest$ mkdir base
gittest$ cd base
base$ ls
base$ echo "abc" > a.txt
base$ echo "def" > b.txt

base$ git init
base git:(master)$ git add *
base git:(master)$ git commit -m 'init'
base git:(master)$ git remote add origin ../base.git
base git:(master)$ git push --set-upstream origin master
```

### 원격지와 연결해야하는 리포지토리로 만들기
```sh
base git:(master)$ rm -rf .git
```

### 데이터를 일부 변경해서 B의 디렉토리 처럼 연출
```sh
base$ echo "hij" > c.txt
base$ echo "abd" > a.txt
```

## 이제 연결 해보기

### base 디렉토리를 git repository로 초기화
```sh
base$ git init
base git:(master)$ git add *
base git:(master)$ git commit -m 'new init'
```

### 리모트 등록
```sh
base git:(master)$ git remote add origin ../base.git
```

### 원격지 데이터 Fetch
```sh
base git:(master)$ git fetch
```

### Branch Tracking 설정
```sh
base git:(master)$ git branch --set-upstream-to=origin/master master
```

### Local와 리모트 자원 병합
```sh
base git:(master)$ git pull origin master --allow-unrelated-histories
```

### 충돌 제거
```sh
base git:(master)$ vi a.txt
base git:(master)$ git commit -m 'merged'
```

### 리모트로 Push
```sh
base git:(master)$ git push
```

## 결과
깔끔하게 원격지 디렉토리와 로컬을 병합하였다.

![](/assets/2019-11-14-git-merge-remote/2019-11-14-git-merge-remote_161635.png)
