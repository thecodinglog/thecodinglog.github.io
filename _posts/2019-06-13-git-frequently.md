---
layout: post
title: 자주쓰는 GIT 명령어 정리
description: 자주쓰는 GIT 명령어 정리
date:   2019-06-13 13:00:00 +0900
author: Jeongjin Kim
categories: git
tags:	git
---
## 합치기
* merge 시작
```plain
git mergetool
```
* Rebase 계속
```plain
git rebase --continue
```
* 선택한 커밋만 합치기. -n 옵션 stage 상태로. 없으면 commit됨
```plain
git cherry-pick -n <커밋id>
```

## 파일 삭제
* Untracked file 과 디렉토리 한번에 삭제. `--dry-run` 옵션 넣어서 지울 파일 한번 확인해보고 하자
```plain
git clean -fd
```

## 되돌리기
* 커밋id로 되돌림. 작업 했던 것은 안 날라감. 최종 수정 했던 상태로 남음
```plain
git reset <커밋id>
```
* 커밋id로 되돌림. 작업했던 것 모두 없어짐
```plain
git reset --hard <커밋id>
```
* 마지막 커밋 취소. 작업했던 것 모두 없어짐
```plain
git reset --hard HEAD
```
* Untracked로 만들기. 변경한것 남음
```plain
git rm --cached <file>
```
* 선택한 파일의 수정사항을 원래대로 되돌림. Stage된것과 새파일은 그대로 있음
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

## 참고사항
### HEAD
마지막 커밋 스냅샷, 다음 커밋의 부모 커밋, 현재 브랜치 마지막 커밋의 스냅샷
### Index
다음에 커밋할 스냅샷
### 워킹 디렉토리
샌드박스
### reset
HEAD를 옮긴다
커밋만 옮김 => git commit 을 취소
```plain
git reset --soft HEAD~
```
커밋을 옮기고 index에 올라간것도 내림 => git commit 과 git add 를 취소
```plain
git reset [--mixed] HEAD~
```
커밋을 옮기고 워킹디렉토리 내용도 지운다
```plain
git reset --hard HEAD~
```

