---
layout: post
title: git에서 트래킹 중인 파일을 로컬에서만 무시하기
description: ignore tracked file locally
date:   2020-04-02 13:00:00 +0900
author: Jeongjin Kim
categories: git
tags:	git gitignore ignore assume-unchanged
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

git이 트래킹하고 있는 파일을 내 로컬에서만 잠시 변경사항을 무시하고 싶다면 어떻게야 할까요?

이런 케이스는 생각보다 많이 있을 수 있습니다. git 리포지토리에 등록된 파일 중에는 소스 파일이 대부분이겠지만
로컬 환경마다 다를 수 있는 설정 파일도 존재하기 때문입니다.

로컬에서 테스트해보기 위해 설정 파일을 건들어야 하는 경우 스테이징 할 때 그 파일만 빼면 되겠지만
언제나 사람은 실수를 하게 되죠.

그래서 그 해당 파일만 변경사항을 숨겨서 commit이 지저분해지지 않게 해봅시다.

간단합니다.

숨기려면

```plain
git update-index --assume-unchanged <파일들>
```

다시 보이게 하려면
```plain
git update-index --no-assume-unchanged <파일들>
```

현재 로컬에서 숨김 처리된 파일들을 보려면

```plain
git ls-files -v | grep "^h"
```

참 쉽죠?