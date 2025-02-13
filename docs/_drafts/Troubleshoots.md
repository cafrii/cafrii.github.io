---
layout: post
title:  Troubleshoots
category:
tags:
---


# 문제 해결

개발 중에 경험한 문제점들 및 해결 방안을 정리합니다.

## https 프로토콜 문제

로컬에서 브라우저로 `127.0.0.1:<port>` 접속 하여 테스트 할 때 jekyll 로그에 다음과 같이 에러 메시지가 나오면, https 대신 http 로 시도해 보세요.
```
[2025-02-12 10:22:38] ERROR bad Request-Line ...
```

## 청소

임시 파일들을 모두 삭제할 필요가 있는 경우, 다음 명령을 사용하세요.
```
jekyll clean
```

## 캐싱 문제 방지

소스 수정을 하다 보면 예상과 다른 이상한 결과가 보이는 경우가 빈번하게 발생합니다. 대부분 그 원인이 명확하지는 않고 _sites/ 폴더 삭제하거나 임시 파일 청소 후 문제가 사라집니다. 브라우저 캐싱과 관련되어 있을 것으로 추정이 됩니다. Chrome 이나 Edge 의 경우 캐싱을 적극적으로 활용하는 것으로 알려져 있습니다. (aggressive caching 사용)

다음과 같이 서버를 실행하여 이 문제를 회피합니다.

```
jekyll serve --force_polling --incremental --livereload
```
설명
- `--force_polling`: 파일 변경 감지를 강제로 활성화
- `--incremental`: 변경된 파일만 다시 빌드하여 속도 최적화
- `--livereload`: 변경 시 자동으로 브라우저 새로고침

<br>

만약 소스 크기가 크지 않다면, 매번 clean 하고 빌드하는 것도 방법일 수 있겠습니다.

```
jekyll clean && jekyll serve --livereload
```

<br>

다음과 같이 캐싱을 비활성화 하는 것도 한가지 방법일 수 있습니다.

- F12 또는 Ctrl + Shift + I로 개발자 도구 열기
- "Network" 탭 이동
- "Disable cache" 체크
- 페이지 새로고침

## collections config 설정

구글링 또는 AI 답변에서, tags 기능을 위해 _config.yml 에 아래와 같이 추가하라는 내용이 보이지만 아래 설정은 site 객체 속성 값이 제대로 만들어지지 않는 심각한 부작용들을 만드는 것으로 확인됩니다. 공식 문서에도 collections 에 아래와 같은 내용이 어떤 의미를 갖는지에 대해 명확한 설명도 없습니다. 쓰지 않는 것이 좋습니다.

```
collections:
  categories:
    output: true
  tags:
    output: true
  posts:
    output: true
```