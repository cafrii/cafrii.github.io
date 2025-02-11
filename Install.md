

# Jekyll로 Site 설치

Jekyll을 이용하여 GitHub Pages 를 통해 게시될 블로그 사이트를 생성하는 작업의 기록입니다.

* 참고 문서
  * GitHub Docs 문서 중 다음 문서를 참고
  * Jekyll 을 사용하여 사이트 설정 -> [Jekyll 을 사용하여 사이트 만들기](
https://docs.github.com/ko/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll#creating-your-site)


이 문서의 내용은 Install.md 의 내용과 유사하지만 약간의 차이가 있습니다.

아래는 작업한 내용의 상세 기술입니다.


### 사전 준비

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

최종 상태
* 현재 맥에 ruby 3.4.1 을 이용할 수 있는 상태가 되어 있습니다.
* `~/.rbenv/versions/3.4.1` 폴더에 설치가 되어 있고 PATH 에도 포함되었습니다.
* 마지막 .zshrc 는 macOS 에서는 zsh 가 기본 shell 이기 때문이며 다른 shell 인 경우 환경에 맞게 수정되어야 합니다.


### 리포지토리 생성

github 에서 로그인 하고, 다음과 같은 빈 리포지토리를 생성했습니다. \
<https://github.com/cafrii/cafrii.github.io>

이 저장소를 맥북의 로컬 저장소에 클론합니다. 이후 이 로컬 작업 사본에서 작업을 진행합니다.

```
git clone https://<git-clone-url>
```

문서에서는 예시로서 docs/ 폴더와 gh-pages 브랜치를 사용하고 있습니다. 여기서는 그냥 디폴트 브랜치의 repo root 에서 작업을 진행합니다.
어떤 브랜치의 어떤 폴더를 게시할 것인지는 나중에 설정에서 변경 가능합니다.

### 프로젝트 생성

문서에 나온 대로 프로젝트를 생성하려 하면 현재 폴더(.)가 비어 있지 않기 때문에 아래와 같이 에러가 발생합니다.
```
$ jekyll new --skip-bundle .
...
 Conflict: <DIR> exists and is not empty.
 Ensure <DIR> is empty or else try again with `--force` to proceed and overwrite any files.
```
현재 폴더에는 .git/ 폴더가 있는데, jekyll에서 .git 폴더는 건드리지 않을 것이라서 강제로 진행합니다.

```
$ jekyll new --skip-bundle --force .
```

다음과 같은 폴더 구조와 파일들이 생성되었습니다.
```
_posts/
    yyyy-mm-dd-welcome-to-jekyll.markdown
_config.yml
.gitignore
404.html
Gemfile
index.markdown
about.markdown
Gemfile
```

### Gemfile 수정

문서에 나온 대로 Gemfile 의 두 부분을 수정합니다.
* jekyll gem 삭제
* github-pages gem 추가

원본
``` gemfile
...
gem "jekyll", "~> 4.4.1"

gem "minima", "~> 2.5"

# gem "github-pages", group: :jekyll_plugins
...
```

수정
``` gemfile
...
# 이 라인은 코멘트 아웃
#-- gem "jekyll", "~> 4.4.1"

gem "minima", "~> 2.5"

# 아래 라인 활성화 하고 수정
gem "github-pages", "~> 232", group: :jekyll_plugins
...
```
gitub-pages gem 설정의 버전 232 자리에는 github-pages 의 의존성 버전 표 <https://pages.github.com/versions/> 를 참고하여 지원하는 최신 버전을 지정해 주면 됩니다.

Gemfile 수정을 저장한 후 bundle 로 패키지를 설치합니다.

``` shell
$ bundle install
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies...
Fetching json 2.10.1
Installing json 2.10.1 with native extensions
Bundle complete! 7 Gemfile dependencies, 98 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

실행 후 Gemfile.lock 이 생성되는데, 이 파일은 .gitignore 에 추가합니다.

### _config 수정

원본
```
title: Your awesome title
email: your-email@example.com
description: >- # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here. You can edit this
  line in _config.yml. It will appear in your document head meta (for
  Google search results) and in your feed.xml site description.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: jekyllrb
github_username:  jekyll
...
```

수정
```
title: <적절하게 수정>
email: <적절하게 수정>
description: <적절하게 수정>
baseurl:
url: "https://<name>.github.io"
...
```

### 로컬에서 테스트

커밋하기 전에 로컬에서 먼저 테스트 합니다.
```
$ bundle exec jekyll serve --port 5000
Configuration file: <path>/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: <path>
       Destination: <path>/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.114 seconds.
 Auto-regeneration: enabled for '<path>'
    Server address: http://127.0.0.1:5000
  Server running... press ctrl-c to stop.
```

기본 포트 4000 을 다른 목적으로 사용중이라서 `--port 5000` 을 지정했습니다.

https://127.0.0.1:5000 에 접속하여 제대로 페이지가 표시되는지 확인합니다.

### 커밋, 푸시

커밋 하고 푸시 합니다.
```
$ git add .
$ git commit -m 'you message'
$ git push origin
```

clone 한 사본이므로 별도로 remote 및 upstream 설정이 필요 없습니다.
만약 `git init .`으로 로컬에서 생성한 경우라면 다음처럼 remote origin 을 추가해야 합니다.
```
$ git remote add origin https://github.com/<user>/<repo>.git
$ git push -u origin <branch>
```

### 퍼블리싱 소스 설정

게시(퍼블리싱)의 소스로 두 가지 옵션 중 선택할 수 있습니다.\
* From Branch
* GitHub Actions workflow

여기서는 간단하게 특정 브랜치의 특정 폴더로 변경 사항이 push 된 경우 게시하도록 하는 옵션을 사용합니다.

* 현재 변경 사항이 적용된 브랜치 이름을 확인합니다. (보통 디폴트라면 main 또는 master)
* GitHub 사이트의 리포지토리 페이지에 들어갑니다. (repo 이름이 <name>.github.io)
* 상단 'Settings' -> 좌측 패널 'Code and automation' 섹션의 'Pages'를 선택합니다.
* 'Build and deployment' 의 Source 를 'Deploy from a branch'를 선택합니다.
* Branch 아래의 두 드롭다운 리스트에서 브랜치와 폴더를 적절하게 선택하고 'Save' 를 클릭합니다.
  * (예: main 과 / 'root' 선택)

### 확인
github 에 의해 호스팅되는 사이트에 방문하여 확인합니다.
