---
layout: post
title: 자주쓰는 GIT 명령어 정리
description: 자주쓰는 GIT 명령어 정리
date:   2019-06-13 13:00:00 +0900
author: Jeongjin Kim
categories: git
tags:	git
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


## 가져오기
* 리모트에 있는 브랜치를 Local로 가져오면서 브랜치 만들기
```plain
git checkout -b <생성할Local브랜치이름> <원격브랜치이름>
```
* 다른 브랜치에서 일부 파일만 가져오기
```plain
git checkout --patch <가져올브랜치이름> <가져올 파일이름>
```
>y-stage

>n-no stage

>a-stage all hunk

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
* untracked로 만들기
```plain
git rm --cached <file>
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

## 올리기
* tag 리모트로 올리기
```plain
git push origin --tags
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


## 리모트 관리
* 히스토리 정리(브랜치를 삭제했는데 남아 있을 때)-이미 리모트에서 삭제된 브랜치를 정리한다
```plain
git remote prune origin
```
* 리모트의 url을 변경
```plain
git remote set-url <remote.name.to.change> <your.new.remote.url>
```

## 비교하기
* difftool을 이용해서 현재 branch와 다른 브랜치 비교
```plain
git difftool <branch>
git difftool <branch> *.jsp
```
* difftool을 이용해서 커밋 간 비교
```plain
git difftool <commit1> <commit2>
```


## 최초 설정
* 이름과 이메일 설정
```plain
git config --global user.name "이름"
git config --global user.email "이메일"
git config user.name // 이름 확인
git config user.email // 이메일 확인
```
* 줄바꿈 문자 설정
```
// windows 이면
git config --global core.autocrlf true
// mac or linux 이면
git config --global input
```
> 운영체제마다 줄바꿈 문자가 다르기 때문에 설정한다. 윈도우에서 깃으로 올릴 때 `cr`를 빼고 가져올 때 자동으로 붙힌다. 맥은 올릴때 `cr`을 빼고 가져올 때는 그대로 가져온다.

* 히스토리 조회를 그래프로 표시
```
git config --global alias.lt "log --oneline --decorate --graph --all --pretty=format:'%C(yellow)%h %C(red)%d%C(reset) %C(bold yellow)%s %C(bold blue)<%an> %C(green)(%ar)%C(reset)'"
```

* 머지툴 설정
```plain
git config --global merge.tool winmerge
```
* Visual Studio Code 를 기본 에디터로 사용
```plain
git config --global core.editor "code --wait"
```
> vs code를 머지툴로 사용하려면 `~/.gitconfig` 파일에서 직접 수정해서 저장하면 편리하다.
> ```
> [core]
>     editor = code --wait
> [diff]
>     tool = default-difftool
> [difftool "default-difftool"]
>     cmd = code --wait --diff $LOCAL $REMOTE
> [merge]
>     tool = vscode
> [mergetool "vscode"]
>     cmd = code --wait $MERGED
>      ```
* git pull 할 때 fast-forward 만 할 수 있도록 설정
```plain
git config --global pull.ff only
```


## 숨기기
* 트랙킹 중인 파일인데 로컬에서만 변경사항을 무시하기
```plain
git update-index --assume-changed [파일들]
```
* 트랙킹 중인 파일인데 로컬에서만 변경사항을 무시한것 해제하기
```plain
git update-index --assume-unchanged [파일들]
```

* 숨긴 파일 보기
```plain
git ls-files -v | grep "^h"
```


## 인증오류
* remote: HTTP Basic: Access denied 발생시 시도해볼만 한 것
```plain
git config --system --unset credential.helper
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
