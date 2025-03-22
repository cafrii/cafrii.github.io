---
layout: post
title:  블로그 초안 글을 로컬에서 미리 확인하기
category: blog
tags: jekyll
---


## 로컬에서 포스트 내용 확인하기

글을 작성한 후 github pages 로 commit/push 하기 전에 일반적으로 거치는 확인 과정을 요약했습니다.

이 내용은 리포지토리의 Install.md 에도 일부 설명되어 있는 내용인데, 블로그 site 에서는 이 파일의 내용을 확인할 수 없습니다. 그래서 딱 필요한 내용만 발췌하여 이 포스트로 정리했습니다.


#### 1.
글은 먼저 _draft/ 폴더에서 작성을 합니다.
vscode 의 미리보기 기능을 이용하여 분할창으로 같이 보면서 작성하면 좀 편합니다.

_drafts/ 아래의 md 파일의 이름은 임의로 사용 가능합니다. yyyy mm dd 형식을 지키지 않아도 됩니다.


#### 2.
터미널에서 가상환경을 준비합니다. 저는 요 블로그 목적으로는 venv를 이용합니다. 이미 필요한 패키지들은 설치되어 있습니다.

```
$ cd <repo-root>

# 저는 이 github repo 안에 _venv 를 준비해 둔 상태입니다.
$ . _venv/bin/activate
```

#### 3.
로컬에서 실제로 jekyll 로 확인합니다. `--drafts` 옵션을 추가해야만 _drafts/ 폴더 안의 초안 상태의 포스트 글을 확인할 수 있습니다.

```
$ cd <repo-root>/docs

$ bundle exec jekyll serve --port 5000 --drafts
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

제 환경에서는 기본 포트 4000 을 다른 목적으로 사용중이라서 `--port 5000` 을 지정했습니다.

`https://127.0.0.1:5000` 에 접속하여 제대로 페이지가 표시되는지 확인합니다. 만약 접속이 잘 안되면 `http://127.0.0.1:5000` 로 시도합니다.

#### 4.

문제가 없으면, 드래프트 파일을 _posts/ 파일로 이동합니다.파일 명은 규칙에 맞게 수정합니다. 프런트매터 부분도 다시 한번 더 확인합니다.

#### 5.

커밋 후 푸시하고, 약간 시간이 지난 후 github site 에서 확인합니다.

끝
