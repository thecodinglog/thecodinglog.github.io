---
layout: post
comments : true
title: git project를 github로 공개하기
---
### git project를 github로 공개하기
#### github에 repository 만들기
- https://github.com/[github 계정명]/[repository 이름].git
- git@github.com:[github 계정명]/[repository 이름].git
#### project를 git project로 만들기
```
>git init
```
#### remote 등록(SSH로 접속)
```
>git remote add [remote명] [리모트주소]
>git remote add origin git@github.com:[github 계정명]/[repository 이름].git
```
#### remote tracking
```
>git branch -u [remote명]/[remote branch명] [local branch명]
>git branch -u origin/master master
```
#### local을 remote로
커밋된 자원을 push
```
>git push
```
>push 할 때
_fatal: refusing to merge unrelated histories_
오류가 발생할 수 있다. 원인은 다양하겠지만 한 가지 예를들면 local에서 git project를 만들고 git hub에서 파일을 하나 생성해서 commit을 해놓으면 두 git project가 공통 시작점이 없기 때문에 발생한다.
이런 경우
>*\>git pull origin branchname --allow-unrelated-histories*
pull 명령 뒤에 옵션을 주어 해결 할 수 있다.
