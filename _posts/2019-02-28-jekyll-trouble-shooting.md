---
layout: post
title: Jekyll - 버전 의존성 문제로 설치가 안될 때
subtitle: 의존성 문제가 깔끔하게 잘 되지 않는 Gem
description: 새로 설치하거나 업데이트 하면 발생할 수 있는 오류 해결 방법
date:   2019-02-28 11:40:00 +0900
author: Jeongjin Kim
categories: ruby
tags:	jetkyll blog troubleshootring
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


# 은근히 시간 많이 걸리는 Jekyll 설치
Jekyll을 사용하기 위해 필요한 환경을 구성하는 것은 Jekyll 홈페이지에 몇 줄 적어놓은
명령어로만 잘 안되는 경우가 많은 것 같다. 윈도우든, 맥이든, 리눅스든 한번 세팅하려고
하면 꼭 어떻게 해야 할지 모르는 오류를 뱉어서 많은 시간을 허비하게 만든다.

Ruby와 그와 관련된 패키지 관리자에 관해서 정확하게 잘 모르고 사용하는 것도 있지만
초보자도 그냥 따라 하면 쉽게 되어야 하는데 의존성 관리가 잘 안 되는 것인지 하나를 해결하면
다른 하나가 안되고 사람 미치게 만드는 무언가 있는 것 같다.

아무튼 얼마 전 사용하고 있는 윈도우 노트북을 포맷하고 나서 Jekyll 환경을 다시 구성하려고
최신버전 Ruby(현재 ruby 2.5.3p105)와 Jekyll을 설치하는데 발생한 오류와 해결한 방법을 적으려고 한다.

# Jekyll 공홈에 있는 설치 방법
1. Ruby 개발 환경 전체 설치
ruby 2.5.3p105버전 Windows RubyInstall 를 이용해서 전체 설치했다.
2. Jekyll과 bundler gem을 설치
```sh
gem install jekyll bundler
```
3. Jekyll 사이트 만들기
```sh
jekyll new myblog
```
3. 새로 만들어진 myblog 디렉터리로 가서 아래 명령어로 서버 실행
```sh
bundle exec jekyll serve
```

이렇게 하면 아무 문제 없이 잘 실행된다.

# 그럼 무슨 문제인가?
문제는 새로 만드는 것이 아니라 기존에 github에 올라가 있던 사이트를 clone 해서 실행한 경우이다. 
받은 디렉터리에서
```
jeklly -v
```
버전 확인만 해도 아래와 같은 오류를 볼 수도 있고
```
Traceback (most recent call last):
        5: from C:/Ruby25-x64/bin/jekyll:23:in `<main>'
        4: from C:/Ruby25-x64/bin/jekyll:23:in `load'
        3: from C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/jekyll-3.8.5/exe/jekyll:11:in `<top (required)>'
        2: from C:/Ruby25-x64/lib/ruby/gems/2.5.0/gems/jekyll-3.8.5/lib/jekyll/plugin_manager.rb:48:in `require_from_bundler'
        1: from C:/Ruby25-x64/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require'
C:/Ruby25-x64/lib/ruby/2.5.0/rubygems/core_ext/kernel_require.rb:59:in `require': cannot load such file -- bundler (LoadError)
```

실행하려고 아래 명령어를 치면
```
bundle exec jekyll serve
```

이런 오류를 볼 수 있다.

```
Traceback (most recent call last):
        2: from C:/Ruby25-x64/bin/bundle:23:in `<main>'
        1: from C:/Ruby25-x64/lib/ruby/2.5.0/rubygems.rb:308:in `activate_bin_path'
C:/Ruby25-x64/lib/ruby/2.5.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
```

# 원인과 해결방법
원인은 알고 보면 간단하다. 다운받은 사이트의 bunlder 버전과 시스템에 설치된 bundler 버전이 맞지 않아서이다.
사이트의 Gemfile.lock 파일을 열어보면 BUNDLED WITH 항목에 버전이 기록되어 있다.
해결 방법은 사이트의 버전을 올리거나, 시스템에 설치된 버전을 사이트에 맞추는 것이다.

## 사이트의 버전 변경
Gemfile.lock 을 삭제하고
```
bundle install
```
을 하면 다시 Gemfile.lock 를 만든다.

## 시스템에 설치된 버전 변경
```sh
gem install bundler -v [설치하고자 하는 bundler 버전]
gem uninstall bundler -v [처음에 설치했던 bundler 버전]
```

# bundler 간단하게 알아보기
애플리케이션 루트에 Gemfile 이라는 이름으로 의존성을 선언한다.
```gemfile
source 'https://rubygems.org'

gem 'rails', '4.1.0.rc2'
gem 'rack-cache'
gem 'nokogiri', '~> 1.6.1'
```

```gemfile
source 'https://rubygems.org'
```

이 줄은 요 사이트에서 gem 선언된 저장소라고 보면 될 것 같다. nexus 같은 jar를 배포할 수 있는 private 저장소를 등록해서 사용하는 것처럼 gem 도 private 저장소가 필요하면 만들어서 참조하게 할 수 있다. 그 밑에 선언된 gem을 하나씩 보면

```gemfile
gem 'rails', '4.1.0.rc2'
```
4.1.0.rc2 버전의 rails gem

```gemfile
gem rack-cache
```
아무 버전 rack-cache gem

```
gem 'nokogiri', '~> 1.6.1'
```
1.6.1 버전과 같거나 크고 1.7.0 보다 작은 버전을 의미하는데 이게 좀 애해하긴 하다.


이렇게 어떤 gem을 쓸 것인지 정의 해놓고
```sh
bundle install
```

하면 정의된 gem 들과 정의한 gem들이 의존하는 모든 gem을 같이 가져온다.(bundle은 bundle install의 단축어이다)

정의한 gem들을 모두 설치하고 난 뒤 설치한 모든 gem 이름과 버전을 Gemfile.lock 이라는 파일에 기록해 놓는다. 기록하는 이유는 패키지가 배포됐을 때 마지막으로 제대로 동작한 모든 gem과 버전을 정확하게 기록해서 어떤 환경에서도 실행할 수 있게 하기 위해서이다.

# 마무리

별것 아닌것에 시간이 더 든다.



