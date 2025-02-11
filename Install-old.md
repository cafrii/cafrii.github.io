

# Jekyll로 Site 설치 (중단)

처음 시도했던 설치 내용이며, 중간에 다른 방법으로 재설치 한 관계로 이 내용은 유효하지 않습니다.

- - -

참조한 문서: \
<https://www.taniarascia.com/make-a-static-website-with-jekyll/>

작업한 내용은 다음과 같습니다.


### 사전 작업

macOS 에서 기본 설치되어 있는 ruby 의 버전은 높지 않아서 brew 패키지로 설치합니다.
``` zsh
# install as brew package
$ brew update
$ brew install rbenv ruby-build
$ which rbenv
/opt/homebrew/bin/rbenv

# install ruby using rbenv
$ rbenv install -l
...
3.4.1
...
$ rbenv install 3.4.1
$ rbenv global 3.4.1
$ ruby -v
3.4.1 (...)

# add path (need to be done only once)
$ echo 'eval "$(rbenv init -)"' >> ~/.zshrc
$ source ~/.zshrc
$ echo $PATH
<MyHome>/.rbenv/shims:...
```

최종적으로, 현재 맥에 ruby 3.4.1 을 이용할 수 있는 상태가 되어 있습니다. `~/.rbenv/versions/3.4.1` 폴더에 설치가 되어 있고 PATH 에도 포함되었습니다. 마지막 .zshrc 는 macOS 에서는 zsh 가 기본 shell 이기 때문이며 다른 shell 인 경우 환경에 맞게 수정되어야 합니다.


### 리포지토리 생성

github 에서 로그인 하고, 다음과 같은 빈 리포지토리를 생성했습니다. \
<https://github.com/cafrii/cafrii.github.io>

이 저장소를 맥북의 로컬 저장소에 클론합니다. 이후 이 로컬 작

### Gemfile 작성

아주 최소한의 내용으로 Gemfile 파일을 작성합니다.
```
gem 'github-pages'
source 'https://rubygems.org'
```


### 번들 설치
```
bundle install
```

Gemfile.lock 파일이 생성되었습니다. 패키지 의존성 목록 같습니다.


### 프로젝트 생성

문서에는 아래 처럼 하라고 나와 있지만...
```
jekyll new startjekyll
```

위와 같이 하면 버전 충돌이 발생합니다.
```
$ jekyll new startjekyll
...
~/.rbenv/versions/3.4.1/lib/ruby/gems/3.4.0/gems/bundler-2.6.3/lib/bundler/runtime.rb:319:in 'Bundler::Runtime#check_for_activated_spec!': You have already activated public_suffix 6.0.1, but your Gemfile requires public_suffix 5.1.1. Prepending `bundle exec` to your command may solve this. (Gem::LoadError)
```

bundle 의 역할이 버전 의존성 조정이라더니, 이제 이해가 갑니다. 다음과 같이 실행해야 합니다. 앞으로 모든 jekyll 실행에는 이렇게 해야 합니다.
```
$ bundle exec jekyll new startjekyll
...
New jekyll site installed in <path>/taniarascia/startjekyll.
```

startjekyll/ 폴더 구조가 생성되었습니다.
```
    _posts/
    _config.yml
    about.md
    index.md
    ...
```

```
$ cd startjekyll
$ bundle install
$ bundle exec jekyll serve --port 5000

<home>/.rbenv/versions/3.4.1/lib/ruby/gems/3.4.0/gems/jekyll-3.10.0/lib/jekyll.rb:26: warning: logger was loaded from the standard library, but will no longer be part of the default gems starting from Ruby 3.5.0.
You can add logger to your Gemfile or gemspec to silence this warning.
<home>/.rbenv/versions/3.4.1/lib/ruby/gems/3.4.0/gems/safe_yaml-1.0.5/lib/safe_yaml/transform.rb:1: warning: base64 was loaded from the standard library, but is not part of the default gems starting from Ruby 3.4.0.
You can add base64 to your Gemfile or gemspec to silence this warning.

bundler: failed to load command: jekyll (<home>/.rbenv/versions/3.4.1/bin/jekyll)
```

역시 3.4.1 대신 3.3.x 를 선택했어야 했나 봅니다. 귀찮음을 피하려면 아무리 스테이블이더라 하더라도 최하위 minor 버전이 1 은 피하는게 옳습니다.

검색 해서 찾은 몇 가지 해결 방안들 중, 프로젝트 폴더 아래에 있는 Gemfile 에 아래 내용을 추가하는 것으로 해결할 수 있습니다.

```
# in ruby 3.4.0, base64 is not included in the standard library
gem 'base64'
gem 'bigdecimal'
```

그 다음 다시 설치하면 됩니다.

```
$ bundle install
$ bundle exec jekyll serve --port 5000
```

커밋 하고 푸시합니다.
```
```