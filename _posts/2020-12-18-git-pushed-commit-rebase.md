---
layout: post
title: 푸시한 커밋을 리베이스 했을 때 브랜치 정리 방법
description: 푸시한 커밋을 리베이스 했을 때 브랜치 정리 방법
date:   2020-12-18 09:00:00 +0900
author: Jeongjin Kim
categories: Git
tags:	git rebase
---

**rebase**는 커밋 그래프를 이쁘게 만들어주는 일등 공신이죠. 저는 항상 rebase를 이용해서 개발 중인 브랜치를 최신화시키는데요
혼자서 사용하는 개발 브랜치면 상관없는데 이게 다른 사람과 공유하면서 작업할 때는 좀 골때리는 상황이 생깁니다.

아래의 경우를 보겠습니다.

![](/assets/2020-12-18-git-pushed-commit-rebase/2020-12-18-git-pushed-commit-rebase_103709.png)

A개발자, B개발자 모두 `master` 브랜치에서 딴 `d` 커밋을 베이스로 작업하고 있었어요. 그런데 A개발자가 `d`커밋을 `master`의 `c`커밋 위로
리베이스를 하고 싶은 겁니다. 아무래도 그편이 그래프가 이쁘게 나오니까요.

![](/assets/2020-12-18-git-pushed-commit-rebase/2020-12-18-git-pushed-commit-rebase_104134.png)

```bash
dev>git rebase master
dev>git push -f
```

이렇게 A개발자는 신나게 그래프 정리를 했습니다. 리베이스를 하면 기존에 있던 `d`와 `e`커밋은 없어지고 새로운 커밋이 각각 생기고 `master` 브랜치의 마지막 커밋 다음에 있게 됩니다.

문제는 B개발자인데요 이 상황에서 B개발자가 `pull`로 병합을 해버리면 애초에 원하는 모습이 안 나오고
아래처럼 더 복잡해진 그래프가 나오게 됩니다.

![](/assets/2020-12-18-git-pushed-commit-rebase/2020-12-18-git-pushed-commit-rebase_105849.png)

이렇게 되지 않으려면 B개발자가 가지고 있는 `dev` 브랜치는 깔끔히 정리된 상태에서 B개발자가 작업한 `f`커밋만 추가로 적용이 되어야 합니다.

B개발자의 `dev` 브랜치를 통째로 날리겠습니다.

```bash
dev>git checkout master
master>git branch -D dev
```

그리고 다시 `dev`브랜치를 내려받습니다.

```bash
master>git checkout dev
```
그러면 A개발자와 똑같은 그래프가 나오게 되겠지요.


![](/assets/2020-12-18-git-pushed-commit-rebase/2020-12-18-git-pushed-commit-rebase_111855.png)

그런 다음 B개발자가 작업했던 커밋만 **cherry-pick** 해옵니다.

```bash
dev>git cherry-pick -n f
```

여기서 `f`는 커밋Id입니다. 현재 그래프에서는 `f`가 보이지 않지만 `git reflog` 명령 등으로 확인할 수 있습니다.

```bash
dev>git add *
dev>git push -u origin dev
```

이렇게 하면 깔끔하게 그래프가 깔끔하게 정리됩니다. 만약 `cherry-pick` 할 커밋이 많다면 `rebase -i` 로 커밋을 먼저 합치는 것도 고려하시는 게 좋을 듯 합니다.

![](/assets/2020-12-18-git-pushed-commit-rebase/2020-12-18-git-pushed-commit-rebase_112127.png)


이 방법은 개발자가 같은 팀이라 **소통이 원활이 잘 되는 상황**에서 해야합니다. 아니라면 서로 `push`, `pull` 하고 난리가 날겁니다. **rebase**는 강력하지만 
항상 조심해야 할 기능합니다.



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