---
layout: post
title: Visual Studio Code를 Markdown Editor로 사용하기
description: Visual Studio Code를 Markdown Editor로 사용하기 jekyll 마크다운
date:   2018-07-25 17:30:00 +0900
author: Jeongjin Kim
categories: tool markdown
tags:	visual studio code markdown visualstudiocode
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


jekyll로 blog를 작성하기 편리하도록 Visual Studio Code 에 플러그인과 약간의 설정변경을 하였다.

개인적으로 마크다운 문서편집기로서 원하는 사항은 3가지 정도이다.
1. markdown 문서 실시간 미리보기
2. 단축키로 텍스트 모양 변경
3. 클립보드에 저장한 스크린샷을 단축키로 붙여놓으면 jekyll의 이미지 저장소인 `/assets` 디렉토리에 post 별로 이미지 자동 저장


## 1. 실시간 미리보기
Visual Studio Code가 업데이트 되면서 기본 사양으로 지원한다. 
미리보기 창을 여는 방법은 텍스트 에디터 우측상단에 돋보기 있는 
아이콘![](/assets/2018-07-25-markdown-editor/2018-07-25-markdown-editor_114418.png)을 클릭하면 된다.

## 2. 단축키로 텍스트 모양 변경
### 플러그인 설치
[mdickin.markdown-shortcuts](https://marketplace.visualstudio.com/items?itemName=mdickin.markdown-shortcuts) 를 설치한다.

### 단축키 변경
![](/assets/2018-07-25-markdown-editor/2018-07-25-markdown-editor_114730.png)
설치를 하면 기본 단축키가 바인딩 되는데 개인적으로 자주쓰는 Inline code 의 단축키가 좀 불편해서 변경을 했다. 
왜냐하면 OS 가 윈도우인 경우 inline code로 씌울 텍스트를 선택하고 <kbd>TAB</kbd> 위에 있는 <kbd>`</kbd>를 누르면 
에디터에서 자동으로 양옆에 <kbd>`</kbd> 가 씌워지는데, Mac OS에서는 한글입력 상태인 경우 <kbd>₩</kbd> 로 대체 되버려서 상당히 불편했기 떄문이다.
또 인라인 코드 블록을 만들기 위해서 플러그인 기본값 단축키를 쓰면 되지만 여전히 불편해서 <kbd>ctrl<kbd> + <kbd>'<kbd> 로 변경했다.

#### 변경 방법
좌측하단 설정버튼을 누르고 바로가기 키를 선택한다.

![](/assets/2018-07-25-markdown-editor/2018-07-25-markdown-editor_120119.png)

toggleInlineCode 를 입력해서 검색하고

![](/assets/2018-07-25-markdown-editor/2018-07-25-markdown-editor_132729.png)

연필모양 편집 버튼을 눌러서 원하는 키 조합으로 변경한다. 


## 3. 이미지 저장
### 플러그인 설치
[mushan.vscode-paste-image](https://marketplace.visualstudio.com/items?itemName=mushan.vscode-paste-image) 를 설치한다.

### 플러그인 설정
좌측하단 설정버튼을 누르고 설정을 선택한다.

![](/assets/2018-07-25-markdown-editor/2018-07-25-markdown-editor_150341.png)

우측 사용자 설정에 아래 설정을 추가한다.
```
"pasteImage.defaultName": "HHmmss",
"pasteImage.namePrefix": "${currentFileNameWithoutExt}_",
"pasteImage.path": "${projectRoot}/assets/${currentFileNameWithoutExt}",
"pasteImage.basePath": "${projectRoot}/_site",
"pasteImage.forceUnixStyleSeparator": true,
"pasteImage.prefix": "/",
"pasteImage.insertPattern": "${imageSyntaxPrefix}/assets/${currentFileNameWithoutExt}/${imageFileName}${imageSyntaxSuffix}",
```

클립보드에 저장된 이미지를 markdown 문서에 붙여넣기(<kbd>option</kbd> + <kbd>command</kbd> + <kbd>v</kbd>)하면
`/assets` 디렉토리 밑에 편집하고 있는 파일명 디렉토리가 생기고 그 밑에 `디렉토리명+HHmmss` 패턴으로 이미지가 저장되며 문서에는 이미지 링크가 걸린다.

이렇게 플러그인 2개 설치, 간단한 설정변경으로 매우 편리하게 마크다운 문서작성을 할 수 있다.