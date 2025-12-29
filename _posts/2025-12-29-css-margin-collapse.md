---
layout: post
title: 시리즈1.CSS 기본 원리와 렌더링 이해-번외-박스겹침현상
description: 박스겹침현상
date:   2025-12-29 10:24:00 +0900
author: Jeongjin Kim
categories: CSS
tags:	CSS BoxModel
---


# 마진 겹침(Margin Collapse): 예상치 못한 동작

마진 겹침은 CSS의 가장 혼란스러운 동작 중 하나입니다. **수직 방향으로 인접한 블록 요소들의 마진이 하나로 합쳐지는 현상**입니다.

### 형제 요소 간 마진 겹침

```html
<!DOCTYPE html>

<head>
    <style>
        .first {
            margin-bottom: 30px;
            background: #ff6b6b;
            height: 30px;
        }
        .second {
            margin-top: 20px;
            background: #4ecdc4;
            height: 30px;
        }
    </style>
</head>

<body>
    <div class="first"></div>
    <div class="second"></div>
</body>
</html>

<!-- 실제 간격 = max(30px, 20px) = 30px (50px가 아님!) -->
```
![](/assets/2025-12-29-css-margin-collapse/2025-12-29-css-margin-collapse_164646.png)
<sub>박스 높이(30px)과 마진 간격이 동일함을 볼 수 있다.</sub>

### 부모-자식 간 마진 겹침

```html
<div class="parent">
    <p class="child">첫 번째 자식</p>
</div>
```

```css
.parent {
    margin-top: 20px;
    /* padding이나 border가 없으면... */
}

.child {
    margin-top: 30px;
}

/* 자식의 margin-top이 부모 밖으로 전달됨 */
/* 결과: 부모 위에 30px 마진 생성 */
```
![](/assets/2025-12-29-css-margin-collapse/2025-12-29-css-margin-collapse_100350.png)
<sub>자식의 margin-top 30px이 부모 마진을 무시한채 부모 밖으로 전달되었다.</sub>

![](/assets/2025-12-29-css-margin-collapse/2025-12-29-css-margin-collapse_100538.png)
<sub>부모 컨테이너가 테두리를 가지고 있으면 부모 마진 20px과 자식 마진 30px이 정상적으로 보인다.</sub>

### 마진 겹침을 방지하는 방법

마진 겹침은 **Block Formatting Context(BFC)**를 생성하면 방지됩니다:

```css
/* 1. 부모에 padding 또는 border 추가 */
.parent {
    padding-top: 1px; /* 또는 */
    border-top: 1px solid transparent;
}

/* 2. overflow 속성 사용 */
.parent {
    overflow: hidden; /* 또는 auto, scroll */
}

/* 3. display 속성 변경 */
.parent {
    display: flow-root; /* 현대적 방법 */
    /* 또는 */
    display: flex;
    /* 또는 */
    display: grid;
}

/* 4. position 속성 */
.element {
    position: absolute; /* 또는 fixed */
}

/* 5. float 속성 */
.element {
    float: left; /* 또는 right */
}
```

### 마진 겹침의 규칙

```css
/* 양수 + 양수 = 큰 값 */
.box1 { margin-bottom: 20px; }
.box2 { margin-top: 30px; }
/* 간격 = 30px */

/* 양수 + 음수 = 합계 */
.box1 { margin-bottom: 30px; }
.box2 { margin-top: -10px; }
/* 간격 = 20px */

/* 음수 + 음수 = 더 작은 값 (절댓값이 큰 값) */
.box1 { margin-bottom: -20px; }
.box2 { margin-top: -30px; }
/* 간격 = -30px */
```
