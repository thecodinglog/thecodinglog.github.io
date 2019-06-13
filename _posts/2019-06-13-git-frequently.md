---
layout: post
title: 자주쓰는 GIT 명령어 정리
description: 자주쓰는 GIT 명령어 정리
date:   2019-06-13 13:00:00 +0900
author: Jeongjin Kim
categories: git
tags:	git
---
## Merge/Rebase
* Merge 툴 설정
```plain
git config --global merge.tool winmerge
```
* merge 시작
```plain
git mergetool
```
* Rebase 계속
```plain
git rebase --continue
```

## 파일 삭제
* Untracked file 과 디렉토리 한번에 삭제. --dry-run 옵션 넣어서 지울 파일 한번 확인해보고 하자
```plain
git clean -fd
```

## 커밋 취소
* 커밋id로 되돌림. 작업 했던 것은 안 날라감. 최종 수정 했던 상태로 남음
```plain
git reset <커밋id>
git reset --hard <커밋id>
```


## 파일 수정 취소
* 선택한 파일의 수정사항을 원래대로 되돌림
```plain
git checkout -- <file>
```
* 수정된 모든 파일을 원래대로 되돌림
```plain
git checkout -- .
```


## 브랜치 삭제
* 리모트 브랜치 삭제
```plain
git push origin --delete <branch>
```
* 로컬 브랜치 삭제
```plain
git branch -d <branch>
```

## 현재 작업중인 것 임시 저장
* 현재 변경 사항 저장
```plain
git stash
```
* 저장했던 것 복원
```plain
git stash pop
```

## 브랜치 조회
* 로컬과 리모트에 있는 모든 브랜치 조회
```plain
git branch -a
```


## 리모트 히스토리 정리(브랜치를 삭제했는데 남아 있을 때)
* 이미 리모트에서 삭제된 브랜치를 정리한다
```plain
git remote prune origin
```

## 최초 설정
* 히스토리 조회를 그래프로 표시
```plain
git config --global alias.lt "log --oneline --decorate --graph --all"
```
* 머지툴 설정
```plain
git config --global merge.tool winmerge
```
