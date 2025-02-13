---
layout: post
title:  "Category 및 Tag 추가하기"
category: jekyll
tags: jekyll liquid
---

## 카테고리 기능 추가하기

Jekyll 을 이용하여 GitHub pages 에 블로그 글을 게시하기 위한 작업을 진행 중입니다.

블로그 메인 페이지에 들어가 보니, 작성한 포스트 들이 시간 순서대로 표시가 되고 있습니다. 포스트 용 markdown 소스의 첫 부분은 Front matter 라고 부르는 yaml 문법의 설정을 적는 곳이 있고, 여기의 categories 와 tags 에 뭔가 적긴 했는데 이에 관한 내용은 보이지 않습니다.

```
---
layout: post
title:  "Markdown 시험"
date:   2025-02-11 10:06:00 +0900
categories: # 바로 이곳에 적은 정보들을 활용하고자 합니다.
tags: # 이것도..
---
```

minima 테마가 정말 minimal 이라서 이런 기능은 제공하지 않는 것 같습니다. 약간의 스터디가 필요한 단계입니다.


## 목적
포스트 별로 기록된 카테고리, 태그를 이용하여 계층적으로 보여주는 기능을 구현하고자 함!

## 어떤 기술을 이용할 것인지 검색

다음과 같이 몇 가지 방법들이 검색 됩니다.
- 테마 가져다 쓰기 또는 수정하기
- 플러그인 활용하기
- javascript 라이브러리 이용하기
- 외부 서비스 연동하기

다른 테마로 바꿔볼 수도 있겠지만 공부도 할겸 minima 테마를 유지하면서 원하는 기능을 추가해 보기로 합니다.

<br>

## Liquid 스터디

### 검색, 구글링

jekyll 문서 페이지에 간단하게 설명은 나와 있습니다. liquid 자체 문서 페이지도 있습니다.

- <https://jekyllrb.com/docs/liquid/>
- <https://shopify.github.io/liquid/basics/introduction/>


### Liquid 기본 문법 숙지
html 페이지 내부에, html 마커와 충돌하지 않는 특수 표식을 이용하여 간단한 로직을 추가하는 방식입니다.
변수, 상수 등을 쓸 수 있고, for, if 등의 간단한 루프, 조건 분기를 지원합니다. 내용 중에서 site 객체를 통해 categories, tags 에 접근 할 수 있습니다.
...


* site 객체
  * Jekyll 사이트의 모든 정보에 접근할 수 있습니다.
  * site.posts 는 모든 포스트의 목록을, site.categories 는 모든 카테고리의 목록을, site.tags 는 모든 태그의 목록을 제공합니다.

이 site 객체를 통해 categories, tags 정보를 얻어와서 sidebar 에 목록을 표시해 주는 간단한 html 을 작성해야 합니다.

더 자세한 내용은 생략합니다. 문제 발생할 때 그 때 참조 문서를 찾아보기로 하고 다음 단계로 진행합니다.


## 포스트에 카테고리 와 태그 적용

각 post 파일 선두에 약간의 메타 정보를 입력하는 부분이 있는데 categories 가 있습니다.

지난 포스트 중 하나의 예시:
```
---
layout: post
title:  "Markdown 연습"
date:   2025-02-11 17:06:00 +0900
categories: markdown
---
```

`category: <카테고리 이름>` 또는 `categories: <카테고리1> <카테고리2>` 와 같이 단수형, 복수형 모두 지원합니다. 복수형을 사용할 경우 공백을 기준으로 구분되므로, 카테고리 이름이 공백을 포함하는 경우 주의가 필요합니다.


## 레이아웃 변경 방법 검토

디폴트 레이아웃을 수정해야 합니다. 기본 레이아웃은 _layouts/default.html 입니다. 스타일은 보통 assets/ 폴더에서 관리합니다.

하지만 현재 소스 트리에는 _layouts/, assets/ 등과 같은 폴더가 아예 없습니다. 그럼 어떻게 보여지고 있는 것일까요?

지금은 `theme: minima` 를 사용 중이고, minima 테마의 기본 레이아웃은 jekyll 의 테마 디렉터리 내부에서 관리되고 있습니다. 그래서 _layouts/ 폴더 없이도 블로그가 정상 동작하고 있는 것입니다.

기본 테마는 그대로 활용하면서 약간의 수정만 하려면, 다음 두 가지 정도의 방법이 있을 것 같습니다.
- (1) 기본 테마 그대로 사용하되, 필요한 부분만 오버라이드 하기
- (2) `theme: minima` 를 제거하고 레이아웃을 직접 관리

방법 1은 기존 테마는 그대로 유지하면서 필요한 부분만 오버라이드 하는 방식이라서, 나중에 테마가 버전 업데이트로 개선이 되면 그 효과도 누릴 수 있다는 장점이 있는 반면, 내가 수정한 부분이 테마 패키지의 다른 파일과 일관성이 깨질 수 있다는 단점도 있습니다.

하지만, 테마 전체를 변경할 계획은 현재로선 없기 때문에, 단점이 있더라도 방법 (1)로 진행하기로 합니다. 향후 버전 업데이트를 고려하여 수정 방식을 신중하게 고민 후 진행하도록 합니다.


## minima 소스 위치 확인

minima 가 설치된 위치를 알아야 합니다. `bundle info` 로 확인 가능합니다.
```
$ bundle info minima
  * minima (2.5.1)
	Summary: A beautiful, minimal theme for Jekyll.
	Homepage: https://github.com/jekyll/minima
	Path: <My-home>/.rbenv/versions/3.4.1/lib/ruby/gems/3.4.0/gems/minima-2.5.1
	Reverse Dependencies:
		github-pages (232) depends on minima (= 2.5.1)
```

>
> 주의: 제대로 된 경로의 bundle 을 실행하고 있는지 확인하세요.
> macOS 내장된 기본 ruby 환경 대신 brew 로 설치된 rbenv 환경을 사용하는 경우라면 아래와 같은 경로의 bundle 이 실행되어야 합니다.
> ```
> $ which bundle
> <your-home>/.rbenv/shims/bundle
> ```

팁: vs code 와 같은 editor 에서 minima 소스를 쉽게 접근하기 위해 다음과 같이 symlink 를 만들어 주었습니다.
```
cd <repo>
mkdir _gems
ln -s ~/.rbenv/versions/3.4.1/lib/ruby/gems/3.4.0/gems/minima-2.5.1 minima-2.5.1
```


## 스타일 오버라이드 전략

style 의 경우 minima 소스는 여러 파일들로 구성되어 있어서, 어떤 파일을 오버라이드 할 지 고민됩니다.

원본 _includes/head.html 의 내용
``` html
<head>
  ...
  <link rel="stylesheet" href="{{ "/assets/main.css" | relative_url }}">
  ...
</head>
```

원본 main.css 및 포함되는 관련 파일들
```
<minima_src> 트리/
    _sass/
        minima/
            _base.scss
            _layout.scss
            _syntax-lighlighing.scss
        minima.scss
    assets/
        main.scss
        minim-social-icons.svg
```
head.html 에서 지정한 assets/main.scss 에서부터 필요한 스타일 파일이 include 되고 있는 구조입니다.

_includes/head.html 을 오버라이드하지 않고,
assets/css/custom.css 를 새로 만들어 필요한 스타일 만 살짝 추가하는 방법을 사용하겠습니다.

- 생각해 볼 수 있는 여러 방안들:
  - _includes/head.html 을 오버라이드
    - 이 방법은 왠지 좋지 않아 보입니다.
  - default.html 에 직접 `<link>` 태그로 추가
    - 이 방법도 왠지 안좋습니다.
  - _sass/ 내의 파일들을 수정 (오버라이드)
    - 향후 minima 가 업데이트 되면서 변화가 발생할 가능성이 큽니다.
  - _config.yml 에서 `style: /assets/css/custom.css` 옵션을 추가
    - 일부 스타일 만 추가할 것이라서, 이 방식이 가장 낫습니다.



## 레이아웃 수정

전략이 결정 되었으니 파일을 수정합니다. 먼저 레이아웃 입니다. 원본 파일을 복사해 옵니다.

```
## copy layout file
$ cd <repo_root>/docs
$ mkdir _layouts
$ cp <minima_src>/_layouts/defualt.html _layouts/
```
- <repo_root> 는 현재 작업 중인 리포지토리의 root 경로 입니다.
- <minima_src> 는 앞서 확인한 minima 테마 패키지의 소스 경로 입니다.



파일을 수정합니다. 원본 파일에서 다음 부분을 추가합니다.
원본 파일에서, `<body>` 는 총 3개 영역으로 나뉘어 있습니다.
header 는 페이지 상단, main 은 실제 컨텐츠가 위치하는 가운데 영역,
footer 는 페이지 아랫 부분을 의미합니다.

{% raw %}
`{.. header ..}` 와 `<main>` 사이에 `<aside>..</aside>` 블럭을 추가합니다.
`<aside>..</aside>` 와 `<main>..</main>` 을 포함하는 새 `<div>` 층을 추가하고 "container" 클래스 스타일을 적용합니다.

``` html
<!DOCTYPE html>
<html lang="{{ page.lang | default: site.lang | default: 'en' }}">
  {%- include head.html -%}
  <body>
    {%- include header.html -%}
    <!-- 여기부터 수정 시작. container div로 한번 감쌉니다. -->
    <div class="container">
      <aside> <!-- sidebar 를 표시하기 위한 div 입니다. -->
      {% include sidebar.html %}
      </aside>

      <main class="page-content" aria-label="Content">
        <div class="wrapper">
        {{ content }}
        </div>
      </main>
    </div>
    <!-- 수정 끝 -->

    {%- include footer.html -%}
  </body>
</html>
```
{% endraw %}


## sidebar.html

그 다음은, site 소스 트리에 _includes/ 폴더를 생성하고, sidebar.html 을 작성합니다.

{% raw %}
includes/sidebar.html
```
<div class="sidebar">
  <h3>카테고리</h3>
  <ul>
    {% for category in site.categories %}
      <li>
        <a href="/categories/{{ category[0] | slugify }}/">
          {{ category[0] }} ({{ category[1] | size }})
        </a>
      </li>
    {% endfor %}
  </ul>

  <h3>태그</h3>
  <ul>
    {% for tag in site.tags %}
      <li>
        <a href="/tags/{{ tag[0] | slugify }}/">
          {{ tag[0] }} ({{ tag[1] | size }})
        </a>
      </li>
    {% endfor %}
  </ul>
</div>
```
{% endraw %}

여기까지만 작업 한 상태에서 jekyll 빌드 후 브라우저로 확인해 보면, 카테고리와 태그 목록이 표시되는 sidebar 가 표시됩니다. 물론 스타일이 적용 안된 관계로 header 와 main 영역 사이에 이상한 위치에서 보입니다.

이상하게 보이는 이 sidebar 요소들이 제대로 보일 수 있도록 이제 스타일을 적용해야 합니다.


## 스타일 수정

앞서 설명한 스타일 수정 전략은, _config.yml 에서 `style: /assets/css/custom.css` 옵션을 추가하는 것이었습니다.

sidebar 에 표시되는 `<div>` 요소에는 "sidebar" 클래스를 지정했으므로, 이 클래스에 대한 스타일을 추가합니다.

`<repo_root>/assets/css/custom.css` 를 작성합니다.

``` css
.container {
    display: flex;
}
aside {
    width: 250px;
    background: #f4f4f4;
    padding: 15px;
    order: 1; /* right sided */
}
main {
    flex-grow: 1;
    padding: 15px;
}
```

`order: 1;` 에 대해서 약간의 설명이 필요한데, flexbox 컨테이너 내에서 기본 순서 (order: 0) 보다 뒤에 배치되도록 하는 것으로서, aside 요소가 main 의 오른쪽 구석에 표시되도록 합니다.


이제 이 custom.css 가 적용될 수 있도록 해야 하는데, 이것도 몇 가지 방법이 있습니다.
1. `<head>` 를 직접 수정
2. _config.yml 에서 지정


1번 방법은 head 요소 내용을 직접 수정하는 것입니다. minima 의 원본 html 에서 head 요소만 살짝 변경하는 방식입니다. 2번 방법은 _config.yml 에 직접 `style: <path>/custom.css` 를 지정하는 방법인데, 이 방법은 더 이상 지원되지 않는 것으로 보입니다.

head 요소가 위치하는 파일은 minima 버전에 따라 다를 수 있습니다. 저의 경우는 head 요소가 default.html 이 아닌 _includes/head.html 에 있었고, 그러다보니 head.html 까지 오버라이드 해야만 했습니다.

_includes/head.html 오버라이드 후 내용 수정 예:
``` html
<head>
...
  <link rel="stylesheet" href="{{ '/assets/main.css' | relative_url }}">
  <link rel="stylesheet" href="{{ '/assets/css/custom.css' | relative_url }}">
...
</head>
```
원본 head.html 에 있던 /assets/main.css 링크 외에 추가로 /assets/css/custom.css 링크를 추가한 것입니다.

이 상태로 빌드 후 확인해 보면 sidebar 가 오른쪽에 제대로 보입니다.

앞서 두 개의 url 경로를 추가했으므로 아래에서 이 경로에 대한 페이지를 작성해야 합니다.

<br>

## 카테고리 페이지

/categories.html 을 작성합니다.

{% raw %}
```
---
layout: default
title: 카테고리
---

<h1>카테고리별 글 목록</h1>

{% for category in site.categories %}
  <h2 id="{{ category[0] | slugify }}">{{ category[0] }}</h2>
  <ul>
    {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
```
{% endraw %}


## 태그 페이지

/tags.html 도 작성합니다.

{% raw %}
```
---
layout: default
title: 태그
---

<h1>태그별 글 목록</h1>

{% for tag in site.tags %}
  <h2 id="{{ tag[0] | slugify }}">{{ tag[0] }}</h2>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
```
{% endraw %}

이제 sidebar 에 표시되는 categery 또는 tag 를 선택하면, main 영역에 선택된 카테고리 또는 태그를 포함하는 포스트 목록이 표시됩니다.


<br>

## 참고
- Liquid 공식 문서: <https://shopify.dev/api/liquid>

## 배운 점
- liquid 동작 원리 이해
- 오버라이딩을 이용한 jekyll 테마의 기능 확장


