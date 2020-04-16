---
layout: post
title: npm install 할 때 예외가 발생될 때
description: 
date:   2020-04-17 08:00:00 +0900
author: Jeongjin Kim
categories: Node.js
tags:	package-lock.json npm Nodejs
---

github와 같은 공개 소프트웨어 소스 사이트에서 Node.js 프로젝트를 clone 한 뒤 `npm install` 을 하면
해당 프로젝트에서 의존하고 있는 패키지를 모두 설치할 수 있습니다.

npm이 사용하는 package.json 파일에는 해당 프로젝트가 의존하고 있는 다른 패키지의 버전 정보를 가지고 있는데 버전 정보가 대부분 범위로 지정이 되어있습니다.
이외 다르게 `package-lock.json` 에는 `package.json` 파일이 생성됐을 때 또는 의존 정보가 변경됐을 때 당시의 버전 정보를 담고 있습니다.
이를 이용해서 다른 사용자가 install을 할 때 버전이 다름으로 인한 오류를 예방할 수 있습니다.

그런데 `npm install` 할 때 예외가 발생하는 경우가 있습니다. 저는 그냥 소스 다운 받고 `npm install` 을 했을 뿐이지만요.


```bash
../fsevents.cc:43:32: error: no template named 'Handle' in namespace 'v8'
    static void Initialize(v8::Handle<v8::Object> exports);
                           ~~~~^
../fsevents.cc:43:32: error: no template named 'Handle' in namespace 'v8'
    static void Initialize(v8::Handle<v8::Object> exports);
                           ~~~~^
In file included from ../fsevents.cc:73:
../src/constants.cc:89:11: warning: 'Set' is deprecated: Use maybe version [-Wdeprecated-declarations]
  object->Set(Nan::New<v8::String>("kFSEventStreamEventFlagNone").ToLocalChecked(), Nan::New<v8::Integer>(kFSEventStreamEventFlagNone));
          ^
In file included from ../fsevents.cc:73:
../src/constants.cc:89:11: warning: 'Set' is deprecated: Use maybe version [-Wdeprecated-declarations]
  object->Set(Nan::New<v8::String>("kFSEventStreamEventFlagNone").ToLocalChecked(), Nan::New<v8::Integer>(kFSEventStreamEventFlagNone));
          ^

이하 생략

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


오류 내역을 보면 뭔가 버전이 안 맞다는 것을 짐작을 할 수 있는데 자신이 설치된 npm 또는 Node.js 버전과
소스를 올린 사람의 버전이 달라서 그럴 수 있겠다는 생각이 들었습니다.

그래서 고정된 `package-lock.json` 을 삭제 후 `npm install` 을 시도했더니 다시 깔끔하게 설치되었습니다.

다만 `package-lock.json` 이 올린 사람의 것과 다르겠지만요...
