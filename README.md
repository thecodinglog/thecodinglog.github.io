# thecodinglog

개발 관련 기술 블로그입니다. Spring, Java, Jekyll 등 다양한 개발 주제를 다룹니다.

## 🚀 로컬 개발 환경 설정

### 1. Ruby 환경 설정 (최초 설정)

이 블로그는 Jekyll을 사용하여 구축되었습니다. Jekyll을 실행하기 위해서는 Ruby 환경이 필요합니다.

#### 1.1 rbenv 설치 (Ruby 버전 관리자)
```bash
# Homebrew로 rbenv 설치
brew install rbenv ruby-build

# rbenv 초기화
rbenv init

# zsh 설정 파일에 rbenv 초기화 추가
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc

# 설정 다시 로드
source ~/.zshrc
```

#### 1.2 Ruby 설치 및 설정
```bash
# Ruby 3.1.4 설치 (Jekyll과 호환성 좋음)
rbenv install 3.1.4

# 전역 Ruby 버전으로 설정
rbenv global 3.1.4

# Ruby 버전 확인
ruby --version
# 출력: ruby 3.1.4p... (2022-07-14 revision a6b64e68c9) [x86_64-darwin21]
```

#### 1.3 Bundler 설치
```bash
# Bundler 설치
gem install bundler

# 설치 확인
bundle --version
```

### 2. 의존성 설치 및 서버 실행

```bash
# 프로젝트 의존성 설치
bundle install

# Jekyll 서버 실행
./start_server.sh
```

서버가 성공적으로 실행되면 `http://localhost:4000`에서 블로그를 확인할 수 있습니다.

## 📝 블로그 포스트 작성

새로운 블로그 포스트는 `_posts/` 디렉토리에 `YYYY-MM-DD-제목.md` 형식으로 작성합니다.

## 🛠️ 기술 스택

- **정적 사이트 생성기**: Jekyll
- **언어**: Ruby
- **스타일**: SCSS
- **배포**: GitHub Pages

## 📚 주요 카테고리

- Spring Framework
- Java 개발
- Jekyll 블로그
- 개발 도구 및 팁
- 클라우드 네이티브 Java

## 🤝 기여하기

블로그 개선이나 오류 수정에 기여하고 싶으시다면 Pull Request를 보내주세요.

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다. 자세한 내용은 [LICENSE.md](LICENSE.md) 파일을 참조하세요.
