---
layout: post
title:  "Markdown 연습"
date:   2025-02-11 17:06:00 +0900
category: jekyll
tags: markdown
---

## 시작
포스트 하나를 작성하려고 하니 막상 마크다운 문법을 알지 못해 예상치 않은 화면이 결과로 나옵니다.

> 마크다운 문법은 별도로 정해진 표준이 없는데, 저의 경우 _config.yml 에 다음과 같이 적용이 되어 있습니다.
> > markdown: kramdown
>
(편집) 중간에 사이트 재생성한 관계로 위 내용은 정확하지 않을 수 있습니다. 하지만 디폴트 설정이 kramdown 이 아닌가 싶습니다. 배운대로 표시되는 걸 보면.....

마크다운의 종류를 선택해야 하는 모양입니다.

> [GirHub Pages 문서](https://docs.github.com/ko/pages/setting-up-a-github-pages-site-with-jekyll/setting-a-markdown-processor-for-your-github-pages-site-using-jekyll):\
> GitHub Pages는 kramdown 및 GitHub의 자체 Markdown 프로세서를 지원하며, 이는 GitHub를 통해 GFM(GitHub Flavored Markdown) 을 렌더링하는 데 사용됩니다. 자세한 내용은 GitHub에서의 쓰기 및 서식 지정 정보을(를) 참조하세요.


현재로선 장단점을 모르니, 그냥 AI의 추천을 받아 결정하기로 합니다.


ChatGPT, Gemini (2.0 flash)의 의견을 요약해 보니 대략 다음과 같습니다.
* kramdown 을 추천. 기능이 풍부하고 GFM 도 포함함.
* kramdown 강점: 수식 표현, HTML 태그 사용 가능, 확장성
* GFM 강점: GitHub 의 READM.md 등에서 쓰는 문법과 100% 동일. 그런데 kramdown 이 GFM 을 포함한다니 차별화 포인트는 아닌 듯.
* 블로그 운영하다가 중간에 마크다운 프로세서를 변경하면 기존 포스트의 레이아웃이 깨질 수 있음. 수작업 변경, HTML로 표시 등 약간의 복원은 가능. 따라서 처음에 잘 결정하는 것이 중요.

그래서, kramdown 으로 결정합니다. 이미 _config.yml 에 설정되어 있는 값이니 따로 수정할 필요도 없습니다.

이제 kramdown 문서를 보면서 자주 사용하는 몇 가지 문법을 익혀야 합니다. 참조한 문서들은 글 맨 아래에 정리되어 있습니다.

숙지한 내용들을 바로 이 글 본문에 연습 삼아 작성했습니다.

___

### 문단 구분

빈 줄 (empty line) 하나가 들어가야 paragraph `<p></p>` 분리가 됩니다.

paragraph (문단) 내에서 줄바꿈을 넣으려면,\
이와 같이 줄끝 공백 두개 또는 백슬래시를 사용합니다.

## 헤더

헤더 앞에는 항상 빈 줄이 필요합니다.\
Setext 와 atx 스타일이 있다는군요.

첫번째 레벨
====

두번째 레벨
----

# H1 의 헤더
# H1 의 두번째 헤더
## H2 헤더
...

`#` 로 시작하는 스타일 (이게 Setext 인지 atx 인지 모르겠으나) 에서는 직전 공란이 필요 없는 것으로 보입니다. 일단 입력 해 보고 예상과 다르면 고치는 방식으로 사용할 거라 암기할 필요 까지는 없습니다.


### 블록인용

라인 선두에 `>` 를 사용합니다.
> 예제 인용문
>
> > 두 번 넣으면 중첩된 인용이 가능.
> > 매 라인 마다 넣어야 합니다.


### 코드 블럭
두 가지 스타일을 지원합니다. \
(1) 공백이나 탭 하나로 들여쓰기 방법 \
(2) 틸드 구분기호 사용 (들여쓰기 불필요)

모든 라인을 들여쓰기 하는 것은 매우 불편하므로, (2)번 만을 사용하겠습니다.

시작 구분 기호의 경우 `~~~ python` 과 같이 언어 이름을 명시하여 문법 강조가 가능합니다.

~~~python
import logging
logging.info("this is sample code block")
~~~

### 수평선

셋 이상의 별표 `***`, 대시 `---`, 또는 밑줄 `___` 을 입력합니다.

---
수평선 예시입니다.

### 리스트

별표 `*`, 플러스 `+`, 대시 `-` 로 시작하는 라인은 리스트 항목입니다. 어떤 종류의 문자인지는 구분하지 않습니다. 두 칸을 들여쓰면 nested 됩니다. 여러 단계의 중첩이 지원됩니다.

```
* kram
+ down
  - jekyll
  * github

1. kram
  1. down
1. jekyll
```
* kram
+ down
  - jekyll
  * github

1. kram
   1. down
1. jekyll


### 테이블
테이블은 당분간 쓸 일은 없을 거 같아서 생략합니다. (나중에 필요 할 때 업데이트)



### 링크

자동 링크는 bracket 괄호로 묶기만 하면 됩니다.

* 예시: <https://k1005.github.io/2022/01/05/kramdown-syntax/>


실제 URL 대신 링크 텍스트를 사용하려면, 텍스트를 대괄호로 묶고 링크 URL 을 괄호로 묶어서 표현합니다.

```
[링크 텍스트](URL)
```

* 텍스트로 표시된 링크의 예시
  * [kramdown 홈페이지 바로가기](http://kramdown.gettalong.org)




### 이미지

이미지는 아래와 같은 형식으로 표현합니다.

`![alt_name](image_path)!`

![이미지](image_path)!

image_path 는 상대 경로, 절대 경로 모두 지원합니다.
사용될 이미지는 어느 폴더에 올려야 하는지는 나중에 실제로 쓰면서 배워야겠습니다.

기본 글 쓸 정도로만 정리했습니다. 시작이니 가볍게..


## 참조한 문서들

* 공식: https://kramdown.gettalong.org/syntax.html
* Quick Ref가 더 유용해 보입니다. https://kramdown.gettalong.org/quickref.html
* 어느분의 번역: https://k1005.github.io/2022/01/05/kramdown-syntax/

## 배운 점
- kramdown 마크다운 syntax

## 배워야 할 점
- image 는 어떻게 올리는지?
