---
layout: post
title: 시리즈1.CSS 기본 원리와 렌더링 이해-1편.CSS는 어떻게 브라우저에서 동작하는가?
description: CSS가 단순한 꾸미기가 아니라 브라우저 렌더링 엔진과 긴밀히 연결된 시스템임을 이해하자
date:   2025-08-29 13:00:00 +0900
author: Jeongjin Kim
categories: CSS
tags:	CSS Rendering
---

HTML에 `<div>` 하나 추가했는데 갑자기 전체 레이아웃이 무너지거나, CSS 속성 하나만 바꿨는데 페이지가 버벅거리기 시작하는 경험이 있으신가요?

무엇 때문에 이런일이 벌어지는지 의문을 품었었다면, 이번 글이 답을 줄 수 있습니다. CSS는 단순히 "꾸미기"가 아니라, 브라우저의 렌더링 엔진과 긴밀히 연결된 하나의 **시스템**입니다. 이 시스템을 이해하면, 왜 어떤 CSS 속성은 성능에 악영향을 주고, 어떤 속성은 괜찮은지 명확하게 알 수 있습니다.

이 포스트는 브라우저가 HTML과 CSS를 어떻게 처리해서 화면에 보여주는지, 그 전체 과정을 파헤쳐보겠습니다.


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


# 브라우저 렌더링 파이프라인: 6단계의 여정

브라우저가 웹페이지를 화면에 그리는 과정은 크게 6단계로 볼 수 있습니다.

```
HTML/CSS 다운로드 → DOM 생성 → CSSOM 생성 → Render Tree → Layout → Paint → Composite
```
> 참고1 : [Critical Rendering Path](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Critical_rendering_path)

> 참고2 : [Render Tree Construction](https://web.dev/articles/critical-rendering-path/render-tree-construction)


각 단계를 자세히 살펴보겠습니다.

![alt text](https://web.dev/static/articles/critical-rendering-path/render-tree-construction/image/dom-cssom-are-combined-8de5805b2061e_856.png)
<sub>이미지 출처: [Render Tree Construction](https://web.dev/articles/critical-rendering-path/render-tree-construction)</sub>

## 1단계: HTML 파싱과 DOM 트리 생성

브라우저는 HTML 문서를 파싱해서 DOM(Document Object Model) 트리를 생성합니다.

```html
<html>
  <head>
    <title>렌더링 테스트</title>
  </head>
  <body>
    <h1>제목</h1>
    <div class="container">
      <p>내용</p>
    </div>
  </body>
</html>
```

위 HTML은 다음과 같은 DOM 트리로 변환됩니다:

```
html
├── head
│   └── title
└── body
    ├── h1
    └── div.container
        └── p
```

DOM 트리는 문서의 구조만 나타내며, 스타일 정보는 포함하지 않습니다.

## 2단계: CSS 파싱과 CSSOM 트리 생성

CSS 파일을 파싱해서 CSSOM(CSS Object Model) 트리를 생성합니다.

```css
body {
  font-size: 16px;
  margin: 0;
}

h1 {
  color: blue;
  font-size: 24px;
}

.container {
  max-width: 800px;
  margin: 0 auto;
}

p {
  line-height: 1.5;
  color: #333;
}
```

이 CSS는 다음과 같은 CSSOM 트리로 변환됩니다.

```
body {
  font-size: 16px;
  margin: 0;
}
├── h1 {
│     color: blue;
│     font-size: 24px; (상속: font-family 등)
│   }
└── .container {
      max-width: 800px;
      margin: 0 auto;
    }
    └── p {
          line-height: 1.5;
          color: #333; (상속: font-size: 16px 등)
        }
```

CSSOM은 CSS 상속 규칙을 적용해서 각 요소의 최종 계산된 스타일을 담습니다:

CSSOM에서 중요한 점은 **상속(Inheritance)**이 일어난다는 것입니다. `body`의 `font-size: 16px`는 모든 하위 요소에 상속되고, `p` 태그는 이를 그대로 물려받습니다.


## 3단계: Render Tree 생성

DOM과 CSSOM을 결합해서 **Render Tree**를 생성합니다. Render Tree는 **실제로 화면에 보여질 요소**들만 포함합니다.

여기서 주목할 점은
- `display: none` 요소는 제외됩니다
- `<head>`, `<script>` 같은 비시각적 요소도 제외됩니다
- `visibility: hidden` 요소는 포함됩니다 (공간을 차지하므로)

```
RenderObject(body, font-size: 16px)
├── RenderObject(h1, color: blue, font-size: 24px)
└── RenderObject(div, max-width: 800px)
    └── RenderObject(p, line-height: 1.5, color: #333)
```

## 4단계: Layout (Reflow) - 위치와 크기 계산

Render Tree가 완성되면, 브라우저는 각 요소의 **정확한 위치와 크기**를 계산합니다. 이 과정을 **Layout** 또는 **Reflow**라고 부릅니다.

```
뷰포트: 1200px × 800px

body: (0, 0, 1200px, 800px)
h1: (0, 0, 1200px, 32px)
div.container: (200px, 32px, 800px, 100px) // 가운데 정렬 계산
p: (200px, 60px, 800px, 24px)
```

Layout 단계에서는 다음과 같은 계산이 이루어집니다:
- 박스 모델 계산 (width, height, padding, border, margin)
- 상대적 단위 해석 (`%`, `em`, `rem` → `px`)
- Flexbox, Grid 레이아웃 계산
- 포지셔닝 (`relative`, `absolute`, `fixed`)

## 5단계: Paint - 실제 그리기

Layout이 끝나면 브라우저는 각 요소를 **실제로 그립니다**. 이 과정을 **Paint** 또는 **Repaint**라고 합니다.

Paint 단계에서는 다음과 같은 스타일들이 적용됩니다:
- 배경 (background-color, background-image)
- 테두리 (border)
- 텍스트와 폰트
- 박스 그림자 (box-shadow)
- 색상 관련 속성

Paint는 일반적으로 여러 **레이어(Layer)**로 나뉘어 진행됩니다:
```
Layer 1: 배경 (background-color, background-image)
Layer 2: 테두리와 아웃라인
Layer 3: 텍스트 내용
Layer 4: 박스 그림자와 특수 효과
```

## 6단계: Composite - 최종 합성

마지막으로 **Composite** 단계에서 GPU가 모든 레이어를 합쳐서 최종 화면을 만듭니다.

```
GPU Layers:
├── Background Layer
├── Content Layer  
└── Effects Layer (transform, opacity 등)
    ↓
Final Screen: 1200px × 800px 픽셀 배열
```

Composite 단계는 GPU의 도움을 받기 때문에 매우 빠릅니다. 그래서 `transform`이나 `opacity` 같은 속성들이 성능상 유리한 이유기도 합니다.

# Reflow vs Repaint: 성능의 핵심

CSS 속성을 변경할 때 어떤 단계부터 다시 실행되는지에 따라 성능이 달라집니다.

**🔴 Reflow를 유발하는 속성들** (Layout → Paint → Composite 모두 발생):

다음 속성들을 변경하면 Layout 단계부터 다시 실행됩니다:

```css
width, height
margin, padding, border
top, left, right, bottom
font-size, line-height
display, float, position
```

**예제:**
```css
.box {
  width: 100px;
}

.box:hover {
  width: 150px; /* Reflow 발생: Layout → Paint → Composite */
}
```

Reflow는 가장 비용이 많이 들어갑니다. **왜 비용이 클까요?** 
- Layout 계산은 해당 요소뿐만 아니라 **부모, 자식, 형제 요소**에도 영향을 줄 수 있습니다.
- 리플로우는 해당 요소의 자식요소와 부모/조상 요소역시 레이아웃 계산을 진행합니다.
- 전체 파이프라인 (Layout → Paint → Composite)을 다시 실행해야 합니다.
따라서 리플로우는 성능 병목의 주범이 될 수 있습니다.


**🟡 Repaint만 유발하는 속성들** (Paint → Composite 발생):

다음 속성들은 Layout 계산 없이 Paint 단계부터 실행됩니다:

```css
background, color
border-style, border-color
box-shadow, outline
visibility
```

**예제:**
```css
.box {
  background: red;
}

.box:hover {
  background: blue; /* Repaint 발생: Paint → Composite */
}
```

**🟢 Composite만 유발하는 속성들** (GPU 레이어에서만 처리):

다음 속성들은 GPU 레이어에서만 처리되어 가장 빠릅니다:

```css
transform
opacity
filter
```

**예제:**
```css
.box {
  transform: translateX(0);
}

.box:hover {
  transform: translateX(50px); /* Composite만 발생 */
}
```

# 개발자 도구로 렌더링 과정 확인하기

Chrome 개발자 도구를 사용해서 렌더링 과정을 시각화할 수 있습니다.

### Rendering 탭 활용

1. `F12` → 우상단 `⋮` → `More tools` → `Rendering`
![](/assets/2025-08-29-css-rendering/2025-08-29-css-rendering_134351.png)

2. 다음 옵션들을 활성화:
   - **Paint flashing**: Paint가 발생하는 영역을 녹색으로 표시
   - **Layout Shift Regions**: Layout이 변경되는 영역 표시



### Performance 탭으로 병목 지점 찾기

1. `F12` → `Performance` 탭
2. 녹화 시작 후 CSS 애니메이션 실행
3. 결과 분석:
   - **Layout**: 노란색 막대 (Reflow 발생)
   - **Paint**: 녹색 막대 (Repaint 발생)
   - **Composite**: 보라색 막대 (합성 처리)

### 실습용 HTML

다음 코드로 직접 테스트해볼 수 있습니다:

```html
<!DOCTYPE html>
<html>
<head>
<style>
.test-box {
  width: 100px;
  height: 100px;
  background: red;
  margin: 20px;
  transition: all 0.3s;
}

.reflow-test:hover {
  width: 150px; /* Reflow 유발 */
}

.repaint-test:hover {
  background: blue; /* Repaint만 유발 */
}

.composite-test:hover {
  transform: scale(1.2); /* Composite만 유발 */
}
</style>
</head>
<body>
  <div class="test-box reflow-test">Reflow Test</div>
  <div class="test-box repaint-test">Repaint Test</div>
  <div class="test-box composite-test">Composite Test</div>
</body>
</html>
```

# 정리

브라우저 렌더링 파이프라인을 이해하면 다음과 같은 이점이 있습니다:

1. **성능 최적화**: 어떤 CSS 속성이 비용이 큰지 알 수 있음
2. **디버깅 효율성**: 문제가 발생하는 단계를 정확히 파악 가능
3. **설계 개선**: 렌더링을 고려한 CSS 설계 가능

**핵심 원칙:**
- Layout을 유발하는 속성 사용 최소화
- 애니메이션은 `transform`, `opacity` 활용
- 불필요한 Reflow/Repaint 방지

다음 글에서는 이러한 렌더링 원리를 바탕으로 CSS의 기본 개념들(선택자, 상속, 우선순위, 박스 모델)을 살펴보겠습니다.

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