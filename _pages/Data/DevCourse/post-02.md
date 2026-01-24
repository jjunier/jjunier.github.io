---
title: "[6기] 데브코스 DE WIL 02 | 데이터 크롤링 및 분석"
tags:
    - DevCourse
    - Data Engineering
    - Crawling
date: "2025-04-14"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표
---
- 웹 페이지의 요소들을 이해하여 **웹 데이터 크롤링에 활용**한다.
- 인터넷 사용자 간의 약속인 **HTTP 프로토콜 통신을 이해**한다.
- 웹 데이터 크롤링 라이브러리인 **BeautifulSoup과 Selenium 기반의 데이터 분석을 수행**한다.

## CSS(Cascading Style Sheets)이란?
---
색상이나 글꼴을 바꾸는 등 문서의 외형을 예쁘게 꾸며주는 '언어'이다.

## JS(JavaScript)이란?
---
문서에 다양한 기능을 만들어주는 '언어'이다.

## HTML(Hyper Text Markup Language)이란?
---
HTML은 웹 브라우저가 이해하고 보여줄 수 있는 문서를 만들기 위한 하나의 '언어'로써 웹 문서를 만들 수 있다.

```html
<!DOCTYPE html>         <!-- 문서의 버전 -->
<html lang="en">        <!-- HTML 문서 시작 선언 및 문서 기본 언어 설정 -->
    <head>              <!-- 문서에 필요한 정보가 기입되는 곳: head 태그 -->
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatiable" content="IE-edge">
        <meta nam="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>     <!-- 문서의 제목 -->
    </head>
    <body>              <!-- 실제 사용자가 눈으로 확인 가능한 문서의 내용이 입력되는 곳: body 태그-->
    </body>
</html>
```

**HTML 기본 문법**
HTML은 **콘텐츠를 가지는 태그**(ex. <div> 콘텐츠 </div>)와 **콘텐츠를 가지지 않는 태그**(ex. <br/>)로 나뉘어 작성된다. 

**콘텐츠를 가지는 태그**는 열리는 태그(시작 태그)와 닫히는 태그(종료 태그)가 하나의 쌍을 이뤄 작성되어야 한다. 

**콘텐츠를 가지지 않는 태그**는 단일 태그 하나만 가지며 셀프 클로징을 가져야 하는 특징이 있다.

**속성과 값**
다음과 같은 HTML의 코드에서의 속성과 값 그리고 콘텐츠는 다음과 같다:

```html
<a href="https://naver.com">네이버 바로가기</a>
```

해당 코드에서 속성은 `href`, 값은 `https://...`이며 콘텐츠는 `네이버 바로가기`로 이루어진다.

### HTML `<HEAD>` 태그
---
`<head>` 태그는 사람 눈에 보이지 않지만 기계는 읽을 수 있는 **문서의 정보**를 정의하는 영역이다. 여기서 태그가 담을 수 있는 정보의 종류는 다음과 같다:

1. 타이틀
2. 메타 데이터
    2-1. 인코딩 정보
    2-2. 문서 설명
    2-3. 문서 작성자
3. CSS, JavaScript
<br/>

**메타 데이터 - 인코딩**
`charset`(character set)은 **문서에서 허용하는 문서의 집합**을 의미한다. charset에 선언된 문서의 집합 규칙에 따라 문서에서 사용할 수 있는 문자가 제한된다. 이에 대부분 전 세계적인 charset 집합인 `UTF-8`을 사용하는게 일반적이다.

### HTML `<STYLE>`, `<LINK>`, `<SCRIPT>` 태그
---
문서 내용인 콘텐츠의 외형에 영향을 주는 태그들이다. 해당 태그들의 활용은 다음과 같다:

- `<style>` 태그는 문서의 헤드 안에서 style을 지정하는 태그로 사용된다. 
- `<link>` 태그는 컨텐츠를 가지지 않는 단일 태그이다.
- `<script>` 태그는 style 태그와 link 태그의 두 기능을 제공하는 태그로, 콘텐츠 방식과 링크 방식으로 나뉜다.

```html
<!DOCTYPE html>         
<html lang="en">        
    <head>              
        <style>     <!-- body의 p 태그 콘텐츠의 색상을 파랑으로 정의 -->
            body {      
                color: blue;
            }
        </style>
        <link rel="stylesheet" href="style.css">
        <script>
            const hello = 'world';
            console.log(hello)
        </script>
        <script src="script.js"></script>
    </head>
    <body>              
    </body>
</html>
```

### HTML `<BODY>` 태그
---
`<BODY>` 태그는 사람 눈에 실제로 보이는 콘텐츠 영역이다. 해당 태그 안에서 여러 개의 태그를 사용하여 웹 문서를 만들고, 해당 웹 문서를 여러 개를 만들어 사용자에게 웹사이트로 제공한다.

### block(블록 레벨 요소)
---
레고 블록처럼 차곡차곡 쌓이고 화면 너비가 꽉 차는 요소로 블록의 크기와 내/외부에 여백을 지정할 수 있고 일반적으로 페이지의 구조적 요소를 나타낸다. 인라인 요소를 포함할 수 있으나, 인라인 요소에 포함될 수는 없다.

**대표적인 블록 레벨 요소**
`<div>`
가장 흔히 사용되는 레이아웃 태그로 단순히 구역을 나누기 위한 태그로써 사용된다.

`<article>`
블로그, 포스트, 뉴스 기사와 같은 독립적인 문서를 전달하는 태그로써 사용된다.

`<section>`
콘텐츠의 구역을 나누는 태그로, 신문지에서 여러 기사가 각자의 구역에서 각자의 정보를 전달하는 의미와 비슷한 역할을 하는 태그로써 사용된다.

### inline(인라인 레벨 요소)
---
블록 요소 내 포함되는 요소로 주로 문장, 단어 같은 작은 부분에 사용되며 한 줄에 나열되는 특징이 있다. 좌/우 여백을 넣는 것만을 허용한다.

**대표적인 인라인 레벨 요소**
`span`
특별한 의미 없이 콘텐츠의 특정 부분을 그룹화하고 스타일을 적용하기 위해 사용된다.

`a`
클릭하면 페이지를 이동할 수 있는 링크 요소를 만들며, href 속성을 사용하여 이동하고자 하는 파일 혹은 URL을 지정할 수 있다. 또, target 속성을 사용하여 이동해야 할 링크를 새 창(_blank), 현재 창(_self) 등 원하는 타겟을 지정할 수 있다.

`strong`
강한 중요성, 심각성, 긴급성을 나타내기 위해 텍스트를 굵게(Bold)로 표시하여 중요한 부분임을 강조하는데 사용된다.

### 레이아웃(Layout)
---
**레이아웃 태그 #1(`<Header>`, `<Footer>`, `<Main>`)**
`<header>`
블로그의 글 제목, 작성일 등의 주요 정보를 담는 태그로 사용된다.

`<footer>`
페이지의 바닥줄에 사용되며 저작권 정보, 연락처 등의 부차적인 정보를 담는 태그로 사용된다.

`<main>`
페이지의 가장 큰 부분으로 사이트의 내용 즉, 주요 콘텐츠를 담는 태그로 사용된다.

**레이아웃 태그 #2(`<section>`, `<article>`, `<aside>`)**
`<section>`
콘텐츠의 구역을 나누는 태그로, 신문지에서 여러 기사가 각자의 구역에서 각자의 정보를 전달하는 의미와 비슷한 역할을 하는 태그로 사용된다.

`<article>`
블로그, 포스트 뉴스 기사와 같은 독립적인 문서를 전달하는 태그로 사용된다.

`<aside>`
문서의 내용에 간접적인 정보를 전달하는 태그로 쇼핑몰의 오른쪽에 따라다니는 “오늘의 상품” 같은 것으로 사용된다.

### 콘텐츠(Contents)
---
**제목 태그(`<h1>` ~ `<h1>`)**
문서 구획 제목을 나타내는 태그로 Heading이라고 부른다. h1부터 h6까지 표현할 수 있으며, h1 태그는 페이지 내에서 '한 번만' 사용되어야 하고, 구획의 순서는 지켜져야 한다.

**문단 태그(`<p>`)**
문서에서 하나의 문단(Paragraph)을 나타내는 태그로 제목 태그와 함께 사용되기도 단독으로 사용되기도 한다.

**서식 태그(`<b>`/`<strong>`, `<i>`/`<em>`, `<u>`, `<s>`/`<del>`)**
`<b>`/`<strong>`
글씨의 두께를 조절할 수 있는 태그이다.
- `<b>`: 의미를 가지지 않고 단순히 굵은 글씨로 변경한다.
- `<strong>`: 굵은 글씨 변경 후 “강조”의 의미를 부여한다.


`<i>`/`<em>`
글씨의 기울기를 조절할 수 있는 태그이다.
- `<i>`: 기울임과 동시에 텍스트가 문단의 내용과 구분되어야 하는 경우 사용할 수 있다.
- `<em>`: 기울임과 내용에 “강조”를 나타낸다.

`<u>`
글씨에 밑줄을 넣고 주석을 가지는 단어임을 알 수 있으며, CSS로 스타일링하여 빨간 밑줄을 넣는 것으로 "오타"를 나타내는 것처럼 사용 가능하고, 단순하게 "밑줄"만 긋는 용도로는 사용 할 수 없다.

`<s>`/`<del>`
글씨에 취소선을 추가할 수 있는 태그이다.
- `<s>`: 단순히 시각적인 취소선만 추가되고 접근성 기기에 취소에 대한 안내는 하지 않다.
- `<del>`: 문서에서 제거된 텍스트를 나타낼 수 있다. `<ins>` 태그를 함께 사용하면 제거된 텍스트 옆에 추가된 텍스트를 표현할 수 있다.

### 멀티 미디어(Multi Media)
---
**이미지 태그(`<img>`)**
문서 내에 이미지를 넣을 수 있는 태그이다. "src" 속성을 사용해 이미지 경로를 넣으면 이미지가 출력된다. "alt"속성을 사용해 이미지 로딩 문제 시 대체 텍스트를 띄울 수 있다.

**이미지 태그(`<figure>`, `<figcaption>`)**
하나의 독립적인 콘텐츠로 분리하고 그에 대한 설명을 넣을 수 있는 태그로, `<figcaption>` 태그를 사용해 콘텐츠의 설명 혹은 범례를 추가할 수 있고 제일 처음이나 제일 아래에 추가해서 사용할 수 있다. 보통의 경우에 이미지를 넣는데 인용문, 비디오/오디오 등 문서의 흐름에 참조는 되지만 독립적으로 분리되어도 되는 내용을 담을 수 있다.

**비디오 태그(`<video>`)**
문서 내에 영상을 첨부할 수 있는 태그로, "src"속성을 사용해 비디오를 문서 내 첨부 가 가능하다. "poster" 속성을 사용해 비디오가 로드되기 전에 포스터를 보여줄 수 있다. 또한, `<source>` 태그를 사용해 여러 타입의 비디오를 제공할 수 있다.

**오디오 태그(`<audio>`)**
문서 내에 소리를 첨부할 수 있는 태그이다. "src" 속성을 사용하여 소리를 문서 내에 첨부할 수 있다. `<source>` 태그를 사용하면 여러 타입의 비디오를 제공할 수 있다. 또한, "controls" 속성을 사용하면 재생/정지 버튼 등이 있는 컨트롤러를 띄울 수 있다.

### 리스트(List)
---
**정렬되지 않은 목록(`<ul>`, `<li>`)**
기본 불릿 형식으로 목록을 그리며, `<ul>` 태그의 자식요소는 `<li>` 태그만 들어와야 한다.

**정렬된 목록(`<ol>`, `<li>`)**
정렬된 목록 태그로, 기본 숫자 형식으로 목록을 그린다. `<li>`태그를 사용하여 목록을 구성할 수 있고 다양한 태그를 포함할 수 있다. `<ol>` 태그의 자식요소는 `<li>` 태그만 들어와야 한다.

**설명 목록(`<dl>`, `<dt>`, `<dd>`)**
설명 목록 태그로, `<dt>` 태그에 사용된 단어 혹은 내용의 설명을 `<dd>` 태그에 작성할 수 있다. 주로 용어사전이나 "키-값"이 있는 쌍의 목록을 나타낼 때 사용한다. `<dt>` 태그를 여러 개 작성하고 하나의 `<dd>` 태그를 작성하는 것으로 여러 개의 용어를 설명할 수 있다.

### 표(Table)
---
**표 생성(`<table>`)**
표를 만드는 태그로, `<tr>` 태그로 행(row)을 구분한다. `<td>` 태그로 열(cell)을 생성한다.

**열 제목 태그(`<th>`)**
`<th>` 태그를 사용하면 셀의 제목을 만들 수 있다.

**제목 그룹 태그(`<thead>`)**
`<thead>` 태그 안에 **"열(cell)"** 제목의 행을 넣음으로써 그룹을 지을 수 있다.

**표 본문 요소 태그(`<tbody>`)**
`<tbody>` 태그 안에 여러 **"열(cell)의 행"**을 넣음으로써 본문 요소를 그룹 지을 수 있다.

**표 바닥글 요소 태그(`<tfoot>`)**
`<tfoot>` 태그 안에 여러 **"열(cell)의 행"**을 넣음으로써 표의 바닥글 요소를 넣을 수 있다.

**표 설명 태그(`<caption>`)**
`<caption>` 태그를 사용하여 **"표가 가진 데이터에 대한 설명"**을 넣을 수 있다.

### 외부 콘텐츠(`<iframe>`)
---
현재 문서 안에 다른 HTML 페이지를 삽입할 수 있는 태그로, 'src' 속성에 원하는 HTML 문서 또는 URL을 넣을 수 있다. 외부 페이지를 불러올 수 있기 때문에 외부 페이지의 영향을 받을 수 있다.

## 인터넷의 약속, HTTP
---
**인터넷과 웹**
`인터넷(Internet)`은 본래 컴퓨터와 컴퓨터를 서로 연결하기 위해 등장한 **네트워크(Network)**에서 출발했다. 초기에는 가까운 거리의 컴퓨터들을 연결하는 형태였고, 이러한 네트워크들을 묶어 **근거리 지역 네트워크(Local Area Network; LAN)**가 만들어졌다. 이후 이 LAN들이 점차 확장되며 전 세계적으로 연결되었고, 오늘날 우리가 사용하는 범지구적 네트워크, 즉 **인터넷(Internet)**이 탄생하게 되었다.

`웹(Web)`은 이러한 인터넷 위에서 동작하는 서비스 중 하나이다. 인터넷이라는 거대한 네트워크 환경 위에서 정보를 주고 받을 수 있는 공간이 바로 **월드 와이드 웹(World Wide Web; WWW)**이다.

**HTTP의 구조**
`HTTP(Hypertext Transfer Protocol)`는 웹 상에서 클라이언트와 서버가 어떤 규칙으로 정보를 주고 받는지 정해 놓은 약속과 같은 것이다. 주소 창에 URL을 입력하고 엔터키를 입력하는 순간, 브라우저는 서버를 향해 **HTTP 요청(Request)**을 보낸다. 그리고 서버는 해당 요청을 처리한 뒤, 그 결과를 **HTTP 응답(Response)** 형태로 다시 클라이언트에게 전달한다.

HTTP 요청에는 서버가 요청을 정확히 이해하고 처리하기 위해 필요한 정보들이 함께 포함된다. 먼저 **Host**는 **요청을 받는 서버의 이름**을 의미하며, **Resource**는 **서버 내의 어떤 자원을 요청**하는 지를 나타낸다. 여기에 Method는 요청의 목적과 방식을 정의하는 요소로써, `데이터를 조회(GET)`할 것인지, `새로 생성(POST)`할 것인지와 같은 동작을 구분한다.

HTTP 요청과 응답은 공통적으로 Header와 Body 구조를 가진다. `Header`에는 **요청을 보낸 주체와 받는 대상, 데이터의 형식, 요청 시각 등과 같은 메타 정보**가 담긴다. `Body`에는 **실제로 전달하고자 하는 내용이 포함**되어, 웹 페이지를 구성하는 HTML 문서나 JSON 데이터 등이 이 영역에 담긴다.

## HTTP 통신 with Python
---
Python에서는 `request` 라이브러리를 사용하면 간단하게 HTTP 요청과 응답을 처리할 수 있다. `request`는 HTTP 요청을 보내고, 서버로부터 받은 응답을 객체 형태로 다룰 수 있도록 도와주는 라이브러리이다.

가장 기본적인 요청 방식은 `GET` 요청이다. `GET`은 서버에게 특정 자원을 요청할 때 사용되며, 일반적으로 웹 페이지를 조회하거나 데이터를 가져올 때 활용된다.

```python
import requests

res = requests.get("https://naver.com")

# 응답 헤더(Header) 확인
res.headers

# 응답 바디(Body) 확인 (일부분만 출력)
res.text[:1000]
```

위 코드에서 `requests.get()`을 통해 서버로 HTTP `GET` 요청을 보내면, **서버는 HTTP 응답을 반환**한다. 이 응답 객체에는 **Header 정보와 Body 정보가 함께 담겨 있으며, res.headers를 통해 응답 헤더를, res.text를 통해 HTML 문서와 같은 응답 본문을 확인**할 수 있다.

반면 `POST` 요청은 서버로 단순히 정보를 요청하는 것이 아닌, **데이터와 함께 전달하여 서버가 특정 작업을 수행하도록 요청**할 때 사용한다. 예시로, 회원가입, 로그인, 데이터 저장과 같은 작업이 해당된다.

```python
import requests

payload = {"name": "Hello", "age": 13}
res = requests.post(
    "https://webhook.site/363c6360-9174-4a11-91b7-71757d5dfd1d"
    data=payload
)

# 상태 코드 확인
res.status_code
```

POST 요청에서는 Body 영역에 데이터(payload)가 포함되며, 서버는 이 데이터를 기반으로 요청을 처리한다. 이때 응답 상태를 나타내는 **HTTP 상태 코드(Status code)**를 통해 요청이 정상적으로 처리되었는지 여부를 확인할 수 있다.

### 윤리적인 웹 스크래핑 크롤링 진행하기
---
웹에서 데이터를 수집하는 방식에는 크게 **웹 스크래핑(Web Scraping)**과 **웹 크롤링(Web Crawling)**이 있다.

웹 스크래핑은 특정한 목적을 가지고 특정 웹 페이지로부터 원하는 정보를 추출하는 행위를 의미한다. 예를 들어 날씨 정보, 주가 데이터, 뉴스 제목과 같은 데이터를 수집하는 작업이 이에 해당한다.

반면 웹 크롤링은 크롤러(Crawler)를 통해 여러 웹 페이지를 **URL을 따라가며 반복적으로 방문하고, 그 정보를 수집하여 색인(Indexing)**하는 과정이다. 대표적인 예시로 검색 엔진이 웹 페이지를 수집하고 정리하는 방식이 있다.

### 올바르게 HTTP 요청하기
---
웹 스크래핑이나 크롤링을 진행할 때 가장 중요한 것은 **기술적으로 가능하다고 해서 항상 허용되는 것은 아니라는 점**이다. 데이터를 수집하긴 전에 다음과 같은 질문을 스스로 던져볼 필요가 있다:

- 이 스크래핑/크롤링은 어떤 목적을 가지고 있는가?
- 과도한 요청으로 서버에 부하를 주지 않는가?
- 해당 사이트의 정책을 위반하고 있지는 않은가?

이를 위해 등장한 개념이 바로 **로봇 배제 프로토콜(Robot Exlusion Protocol; REP)**이다. 웹 브라우징은 사람이 직접 수행할 수도 있지만, 로봇(프로그램)에 의해 자동으로 수행될 수도 있다. 모든 로봇이 모든 웹 페이지에 접근하는 것이 정당하지 않기 때문에, 웹 사이트는 `robots.txt` 파일을 통해 로봇의 접근 범위를 정의할 수 있다.

```text
# 모든 user-agent에 대해서 접근을 거부
User-agent: *
Disallow: /

# 모든 user-agent에 대해서 접근을 허가
User-agent: *
Allow: /

# 특정user-agent에 대해서 접근을 불허
User-agent: MussBot
Disallow: /
```

이러한 규칙을 통해 사이트 운영자는 어떤 로봇이, 어떤 경로에 접근할 수 있는지를 명시할 수 있다. 윤리적인 웹 스크래핑과 크롤링이란, 단순히 데이터를 가져오는 것을 넘어 **서버 자원을 존중하고, 서비스 제공자의 의도를 존중하는 태도**에서 출발한다.

## 웹 브라우저가 HTML을 다루는 방법
---
웹 브라우저는 단순히 HTML 문서를 받아 화면에 그대로 출력하지 않는다. 브라우저는 서버로부터 HTML 문서를 응답받은 뒤, 내부의 **렌더링 엔진(Rendering Engine)**을 통해 문서를 해석하고 구조화하는 과정을 거친다. 이 과정의 핵심 결과물이 바로 **DOM(Document Object Model)**이다.

![DOM](/assets/img/contents/dom.png "Document Object Model")

브라우저 렌더링 엔진은 HTML 문서를 로드한 후, 문서를 위에서 차례대로 **파싱(Parsing)**한다. 이때 HTML 문서는 단순한 문자열이 아닌, **트리 구조(Tree Structure)**로 변환되며 각 요소는 하나의 객체로 관리된다. 이렇게 생성된 문서 구조를 **DOM**이라고 부른다.

DOM은 HTML 문서를 객체(Object)의 집합으로 표현한 모델이다. 실제 DOM 구조는 매우 복잡하지만, 각 태그를 하나의 노드(Node)로 생각하면 문서를 훨씬 직관적으로 이해할 수 있다. 이 덕분에 브라우저는 문서를 단순히 "보여주는 대상"이 아니라, 조작 가능한 객체 구조로 다룰 수 있게 된다.

브라우저는 먼저 렌더링 과정을 통해 DOM을 생성한 뒤, **DOM Manipulation**, 즉 DOM 조작을 수행할 수 있다. 예를 들어 자바스크립트를 통해 새로운 요소를 추가하거나, 기존 요소를 수정·삭제하는 작업이 가능하다.

```javascript
var imageElement = document.createElement("img");
document.body.appendChild(imgElement);
```

위 코드는 DOM Tree를 순회하여 새로운 `img` 요소를 생성하고 이를 문서의 body에 추가하는 예시이다. 이처럼 **DOM Tree를 기반으로 특정 요소를 추가할 수도 있고, 탐색**할 수도 있다.

```javascript
document.getElementsByTagName("h2");
```

브라우저가 HTML을 그대로 다루지 않고 DOM으로 변환하는 이유는 명확한데, DOM을 사용하면 원하는 요소를 동적으로 변경할 수 있고, **특정 요소를 쉽게 탐색**할 수 있기 때문이다.

### 스크래핑 관점에서 DOM이 주는 인사이트
---
웹 스크래핑 관점에서 보면, 중요한 사실 하나를 알 수 있다. **브라우저는 HTML을 기반으로 DOM을 생성**하고, 이후 **모든 조작과 탐색은 DOM을 기준으로 이루어진다**는 점이다.

즉, 우리가 웹 페이지에서 보고 있는 데이터는 HTML 문자열 그 자체가 아니라, **HTML이 파싱되어 만들어진 DOM 구조의 결과물**이다.
이 관점에서 보면, 파이썬으로 웹 페이지를 분석하기 위해서는 HTML을 DOM과 유사한 구조로 변환해 줄 도구, 즉 **HTML Parser가 필요하다**는 결론에 도달한다.

이 역할을 수행하는 대표적인 라이브러리가 바로 `BeautifulSoup`이다.

## HTML을 분석해주는 BeautifulSoup
---
기본적으로 `requests` 라이브러리를 사용하면 서버로부터 HTML 문서를 문자열 형태로 받아올 수 있다.

```python
import requests

res = requests.get("https://example.com")
res.text
```

하지만, 이 상태의 HTML은 단순한 문자열이기 때문에, 특정 태그를 찾거나 구조적으로 분석하기는 불편하다. 이때 BeautifulSoup을 사용하면 **HTML 문서를 파싱하여 DOM과 유사한 구조로 변환**할 수 있다.

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(res.text, "html.parser")
soup.prettify()
```

BeautifulSoup을 통해 생성된 `soup` 객체는 HTML 문서를 트리 구조로 다룰 수 있게 해준다. 이를 통해 문서의 특정 영역을 쉽게 가져올 수 있다.

```python
# head 가져오기
soup.head

# body 가져오기
soup.body
```

또한 태그 기반 탐색도 매우 직관적으로 수행할 수 있다.

```python
# <h1> 태그 하나 찾기
soup.find("h1")

# <p> 태그 모두 찾기
soup.find_all("p")
```

찾아낸 태그 객체는 이름, 속성, 내용 등을 각각 분리해서 다룰 수 있다.

```python
h1 = soup.find("h1")

# 태그 이름
h1.name

# 태그 내부 텍스트
h1.text
```

이러한 방식은 브라우저가 DOM을 다루는 방식과 매우 유사하며, 스크래핑에 있어 핵심적인 접근법이다.

### 원하는 요소 가져오기 - 책 제목 스크래핑
---
실제 예제를 통해 원하는 데이터를 추출한다. 해당 예시는 책 목록 페이지에서 책 제목 정보를 스크래핑하는 간단한 예제이다.

```python
import requests
from bs4 import BeautifulSoup

res = requests.get(
    "https://books.toscrape.com/catalogue/category/books/travel_2/index.html"
)
soup = BeautifulSoup(res.text, "html.parset")
```

페이지 구조를 살펴보면, 각 책의 제목은 `<h3>` 태그 내부에 포함되어 있다. 이를 기반으로 모든 `<h3>` 태그를 가져온 뒤 반복문을 통해 책 제목을 추출할 수 있다.

```python
h3_result = soup.find_all("h3")

for book in h3_result:
    print(book.a["title"])
```

이처럼 스크래핑의 핵심은 HTML 구조를 먼저 관찰하고, 그 구조에 맞춰 DOM Tree를 탐색하듯 요소를 찾아내는 것이다.

## HTML의 Locator로 원하는 요소 찾기
---
웹 페이지에서 원하는 데이터를 정확히 가져오기 위해서는, 단순히 태그 이름만으로는 한계가 있다. 실제 HTML 문서에는 같은 태그가 수십, 수백 개 존재하기 때문에, **어떤 요소를 대상으로 삼을 것인지 명확하게 지정하는 기준**이 필요하다. 이때 사용하는 것이 바로 **Locator**다.

HTML에서 가장 대표적인 Locator는 `id`와 `class` 속성이다.
`id`는 문서 내에서 고유한 값을 가지며, `class`는 **여러 요소가 공통으로 가질 수 있는 분류 값**이다. 이 두 속성을 활용하면 특정 영역의 데이터를 훨씬 정밀하게 추출할 수 있다.

### 특정 요소(id와 class)를 지정하여 정보 가져오기
---
먼저 기본적인 흐름은 동일하다. `requests`로 HTML 문서를 가져오고, BeautifulSoup으로 파싱한 뒤 원하는 요소를 탐색한다.

```python
import requests
from bs4 import BeautifulSoup

res = requests.get("https://example.python-scraping.com")
soup = BeautifulSoup(res.text, "html.parser")
```

가장 단순한 방식으로는 태그 이름만을 이용하여 요소를 찾을 수 있다.

```python
# id 없이 div 태그 하나 찾기
soup.find("div")
```

하지만 특정 영역을 정확히 지정하고 싶다면 `id`나 `class`를 함께 사용해야 한다.

```python
# id가 "results"인 div 태그 찾기
soup.find("div", id="results")

# class가 "page-header"인 div 태그 찾기
find_result = soup.find("div", "page-header")
```

이렇게 찾아낸 결과는 BeautifulSoup의 태그 객체이며, 내부의 텍스트를 그대로 출력하면 줄바꿈(`\n`)이나 공백이 함께 포함될 수 있다. 이 경우 `strip()`을 사용하여 깔끔하게 정리가 가능하다.

```python
# <h1> 태그 내부 텍스트만 깔끔하게 추출
find_result.h1.text.strip()
```

해당 과정은 브라우저에서 개발자 도구를 특정 영역을 클릭하고, 해당 요소의 id나 class를 확인한 뒤 이를 코드로 옮기는 방식과 동일하다.

### 원하는 요소 가져오기 - Hashcode 질문 스크래핑
---
일부 웹 사이트는 User-Agent가 없는 요청을 비정상적인 접근으로 판단하여 응답을 제한한다. 따라서 브라우저에서 접속한 것처럼 보이도록 User-Agent를 함께 설정하는 것이 중요하다.

```python
user_agent = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) \
    AppleWebKit/537.36 (KHTML, like Gecko) \
    Chrome/83.0.4103.97 Safari/537.36"
}

import requests
from bs4 import BeautifulSoup

res = requests.get("https://hashcode.co.kr/", headers=user_agent)
soup = BeautifulSoup(res.text, "html.parser")
```

페이지 구조를 살펴보면, 질문 목록은 특정 li 태그와 class 조합으로 구성되어 있다. 이를 기반으로 질문 제목을 추출할 수 있다.

```python
questions = soup.find_all("li", "question-list-item")

for question in questions:
    print(question.find("div", "question").find("div", "top").h4.text)
```

많은 웹 페이지는 데이터를 여러 페이지로 나누어 제공하는 페이지네이션(Pagination) 방식을 사용한다. 이 경우 URL의 패턴을 분석해 반복적으로 요청을 보내는 방식으로 데이터를 수집할 수 있다.

```python
import time

for i in range(1, 6):
    res = requests.get(
        "https://hashcode.co.kr/?page={}".format(i),
        headers=user_agent
    )
    soup = BeautifulSoup(res.text, "html.parser")
    
    questions = soup.find_all("li", "question-list-item")
    for question in questions:
        print(question.find("div", "question").find("div", "top").h4.text)
    
    # 서버 부하 방지를 위한 딜레이
    time.sleep(0.5)
```

## 정적 웹 사이트와 동적 웹 사이트
---
지금까지의 웹 스크래핑은 비교적 단순한 HTML 구조를 전제로 했다. 하지만 실제 서비스 환경에서는 HTML이 항상 고정되어 있지 않다. 웹 페이지는 **어떻게 생성되느냐**에 따라 크게 두 가지 유형으로 나뉜다.

**정적(static) 웹사이트**는 서버가 응답할 때 이미 완성된 HTML 문서를 전달한다. 이 경우 브라우저는 HTML을 그대로 렌더링하기만 하면 되며, `requests`와 `BeautifulSoup`만으로도 원하는 정보를 충분히 추출할 수 있다.

반면 **동적(dynamic) 웹사이트**는 서버 응답 이후에도 HTML 내용이 변한다. 초기 응답에는 뼈대만 전달되고, 실제 데이터는 이후에 추가로 채워지는 구조다. 이 과정에서 **렌더링이 완료될 때까지의 지연 시간이 발생**하며, 단순한 HTTP 요청만으로는 완전한 데이터를 얻기 어려운 상황이 생긴다.

### 동적 웹 사이트의 동작 방식
---
웹 브라우저 내부에서는 **JavaScript(JS)**라는 프로그래밍 언어가 실행된다. 동적 웹사이트는 이 JavaScript를 이용해 서버와 추가 통신을 수행하고, 필요한 데이터를 화면에 채워 넣는다.

이때 중요한 개념이 **동기 처리**와 **비동기 처리**다.

- 동기 처리에서는 요청을 보낸 뒤 응답이 올 때까지 기다리므로, HTML 로딩에 문제가 없다.
- 비동기 처리에서는 요청과 응답이 분리되어 실행되기 때문에, HTML이 완전히 렌더링되기 전에 데이터를 추출하면 불완전한 결과를 얻게 될 수 있다.

즉, 동적 웹사이트에서는 “HTML을 받았다”는 사실이 곧 “데이터가 준비되었다”는 의미가 아니다.

## 스크래퍼의 한계점
---
지금까지 사용한 `requests` 기반 스크래퍼는 다음과 같은 한계를 가진다.

첫째, 비동기 처리 환경에서는 서버 응답 직후 데이터를 가져오면 아직 로딩되지 않은 상태의 HTML을 얻게 된다. 이를 해결하려면 임의의 시간을 지연한 뒤 데이터를 가져오는 방식이 필요하지만, 이는 안정적인 해결책이 아니다.

둘째, 키보드 입력이나 마우스 클릭과 같은 UI 상호작용은 `requests`로 처리할 수 없다. 실제 사용자처럼 버튼을 누르거나 입력창에 값을 넣기 위해서는 웹 브라우저 자체를 자동으로 조작해야 한다.

이러한 문제를 해결하기 위해 등장한 도구가 바로 `Selenium`이다.

## 브라우저 자동화 도구, Selenium
---
`Selenium`은 **웹 브라우저를 실제 사용자처럼 조작할 수 있게 해주는 라이브러리**다. 단순한 HTTP 요청이 아니라, **브라우저를 띄우고, 렌더링이 끝난 화면을 기준으로 요소를 다룬다**는 점이 핵심이다.

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.implicitly_wait(10)
driver.get("https://example.com")

# UI와 상호작용 가능
elem = driver.find_element_by_tag_name("hello-input")
elem.send_keys("Hello!")
```

이 방식은 동적 웹사이트에서도 렌더링이 완료된 이후의 DOM을 기준으로 데이터를 추출할 수 있게 해준다.

### Selenium 시작하기
---
Selenium은 브라우저 드라이버가 필요하며, `webdriver-manager`를 사용하면 이를 자동으로 관리할 수 있다.

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

with webdriver.Chrome(service=Service(ChromeDriverManager().install())) as driver:
    driver.get("https://example.com")
    print(driver.page_source)
```

`page_source`를 통해 브라우저가 렌더링을 마친 이후의 HTML을 확인할 수 있다는 점이 중요하다.

**Driver에서 특정 요소 추출하기**
Selenium에서는 다양한 기준을 통해 요소를 탐색할 수 있다. 가장 기본적인 방식은 태그 단위 탐색이다.

```python
from selenium.webdriver.common.by import By

driver.find_element(By.TAG_NAME, "p")
driver.find_elements(By.TAG_NAME, "p")
```

BeautifulSoup이 **정적인 HTML 파서**라면, Selenium은 **실시간 DOM을 대상으로 한 탐색 도구**라고 볼 수 있다.

### Wait and Call
---
동적 웹사이트에서 가장 중요한 개념 중 하나는 **기다림(Wait)**이다.
요소가 생성되기 전에 접근하면 오류가 발생하기 때문에, Selenium은 두 가지 대기 방식을 제공한다.

**Implicit Wait / Explicit Wait**
Implicit Wait는 페이지 내 요소가 모두 로딩될 때까지 **지정한 시간만큼 기다리는 방식**이다.

```python
from selenium.webdriver.support.ui import WebDriverWait

with webdriver.Chrome(service=Service(ChromeDriverManager().install())) as driver:
    driver.get("https://indistreet.com/live?sortOption=startDate%3AASC")
    driver.implicitly_wait(10)
    print(
        driver.find_element(
            By.XPATH,
            '//*[@id="__next"]/div/main/div[2]/div/div[4]/div[1]/div[1]/div/a/div[2]/p[1]'
        ).text
    )
```

반면, Explicit Wait는 **특정 요소가 등장할 때까지 기다리는 방식**으로, 훨씬 정밀한 제어가 가능하다.

```python
from selenium.webdriver.support import expected_conditions as EC

with webdriver.Chrome(service=Service(ChromeDriverManager().install())) as driver:
    driver.get("https://indistreet.com/live?sortOption=startDate%3AASC")
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located(
            (
                By.XPATH,
                '//*[@id="__next"]/div/main/div[2]/div/div[4]/div[1]/div[1]/div/a/div[2]/p[1]'
            )
        )
    )
    print(element.text)
```

**마우스 이벤트 처리하기**
동적 웹사이트에서는 버튼 클릭과 같은 마우스 이벤트가 필수적인 경우가 많다. Selenium의 ActionChains를 이용하면 이러한 이벤트를 처리할 수 있다.

```python
from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
driver.get("https://hashcode.co.kr/")
driver.implicitly_wait(0.5)

button = driver.find_element(By.CLASS_NAME, "nav-link.nav-signin")
ActionChains(driver).click(button).perform()
```

**키보드 이벤트 처리하기**
키보드 입력 역시 사용자 행동을 그대로 재현할 수 있다. 이를 통해 로그인과 같은 절차도 자동화가 가능하다.

```python
from selenium import webdriver
from selenium.webdriver import ActionChains, Keys
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager
import time

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
driver.get("https://hashcode.co.kr")
time.sleep(1)

button = driver.find_element(By.CLASS_NAME, "nav-link.nav-signin")
ActionChains(driver).click(button).perform()
time.sleep(1)

id_input = driver.find_element(By.ID, "user-email")
ActionChains(driver).send_keys_to_element(id_input, "여러분의 아이디").perform()
time.sleep(1)

pw_input = driver.find_element(By.ID, "user-password")
ActionChains(driver).send_keys_to_element(pw_input, "여러분의 비밀번호").perform()
time.sleep(1)

login_button = driver.find_element(By.ID, "btn-sig-in")
ActionChains(driver).click(login_button).perform()
time.sleep(1)
```

## 시각화 라이브러리, Seaborn
---
문자열이나 숫자의 나열만으로는 패턴이나 경향을 파악하기 어렵기 때문에,
**시각화는 데이터 분석의 선택이 아니라 필수**에 가깝다.

지금까지 우리는 다양한 기법으로 데이터를 수집할 수 있었다. 하지만 스크래핑 결과가 여기저기 흩어져 있다면, 그 가치를 제대로 전달하기 어렵다.

이때 시각화는 데이터를 **"보는 사람에게 떠먹여 주는 도구"** 역할을 한다.

`seaborn`은 matplotlib을 기반으로 만들어진 **고수준(high-level) 시각화 라이브러리**다. 복잡한 설정 없이도 다양한 그래프를 손쉽게 그릴 수 있다는 점이 큰 장점이다.

```python
import seaborn as sns

tips = sns.load_dataset("tips")

sns.relplot(
    data=tips,
    x="total_bill", y="tip", col="time",
    hue="smoker", style="smoker", size="size",
)
```

간단한 리스트 데이터만으로도 그래프를 그릴 수 있다.

```python
import seaborn as sns

# Scatterplot을 직접 그려봅시다
# 값 x=[1, 3, 2, 4]
# 값 y=[0.7,0.2,0.1,0.05]

sns.lineplot(x=[1, 3, 2, 4], y=[4, 3, 2, 1])
```

|Line Chart 1|
|---|
|![chart01](/assets/img/contents/chart/chart1.png "Line Chart")|

```python
# Barplot을 직접 그려봅시다
# 범주 x=[1,2,3,4]
# 값 y=[0.7,0.2,0.1,0.05]

sns.barplot(x=[1,2,3,4],y=[0.7,0.2,0.1,0.05])
```

|Bar Chart 1|
|---|
|![chart02](/assets/img/contents/chart/chart2.png "Bar Chart")|

matplotlib과 함께 사용하면 제목, 축 이름, 범위 등을 더욱 세밀하게 제어할 수 있다.

```python
# matplotlib.pyplot을 불러와봅시다.

import matplotlib.pyplot as plt

# 제목을 추가해봅시다.

sns.barplot(x=[1,2,3,4], y=[0.7, 0.2, 0.1, 0.05])
plt.title("Bar Plot")
plt.show()

# xlabel과 ylabel을 추가해봅시다.

sns.barplot(x=[1,2,3,4], y=[0.7, 0.2, 0.1, 0.05])
plt.xlabel("X label")
plt.ylabel("Y label")
plt.show()
```

|Bar Chart 2|Bar Chart 3|
|---|---|
|![chart03](/assets/img/contents/chart/chart3.png "Bar Chart")|![chart04](/assets/img/contents/chart/chart4.png "Bar Chart")|

```python
# lineplot에서 ylim을 2~3으로 제한해봅시다.

sns.lineplot(x=[1,3,2,4], y=[4,3,2,1])
plt.ylim(0, 10)

plt.show()

# 크기를 (20, 10)으로 지정해봅시다.

sns.lineplot(x=[1,3,2,4], y=[4,3,2,1])
plt.figure(figsize=(20, 10))

plt.show()
```

|Line Chart 2|Line Chart 3|
|---|---|
|![chart05](/assets/img/contents/chart/chart5.png "Line Chart")|![chart06](/assets/img/contents/chart/chart6.png "Line Chart")|

## 스크래핑 결과 시각화 ① – 날씨 데이터
---
**Selenium으로 데이터 수집**
동적 웹사이트의 데이터를 수집하기 위해 Selenium을 사용해 기상청 날씨 데이터를 가져온다.

```python
# 스크래핑에 필요한 라이브러리를 불러와봅시다.

from selenium import webdriver
from selenium.webdriver import ActionChains
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.actions.action_builder import ActionBuilder
from selenium.webdriver import Keys, ActionChains
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

# driver를 이용해 기상청 날씨 데이터를 가져와봅시다.

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
driver.get("https://www.weather.go.kr/w/weather/forecast/short-term.do")
driver.implicitly_wait(1)

temps = driver.find_element(By.ID, "my-tchart").text
temps = [int(i) for i in temps.replace("℃", "").split("\n")]
```

수집한 데이터를 바탕으로 꺾은선 그래프를 그려본다.

```python
# 받아온 데이터를 통해 꺾은선 그래프를 그려봅시다.
# x = Elapsed Time(0~len(temperatures)
# y = temperatures

import seaborn as sns

sns.lineplot(
    x = [i for i in range(len(temps))],
    y = temps
)
```

```python
# 받아온 데이터를 통해 꺾은선 그래프를 그려봅시다.

import matplotlib.pyplot as plt

plt.ylim(min(temps) - 5 , max(temps) + 5)
plt.title("Expected Temperature from now on")

sns.lineplot(
    x = [i for i in range(len(temps))],
    y = temps
)

plt.show()
```

## 스크래핑 결과 시각화 ② – 해시코드 질문 태그
---
**질문 태그 빈도 분석**
해시코드 질문 페이지에서 태그 빈도를 수집하고 시각화한다.

```python
# 다음 User-Agent를 추가해봅시다.

user_agent = {"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36"}
# 필요한 라이브러리를 불러온 후, 요청을 진행해봅시다.
# 응답을 바탕으로 BeautifulSoup 객체를 생성해봅시다.
# 질문의 빈도를 체크하는 dict를 만든 후, 빈도를 체크해봅시다.

import time

frequency = {}

import requests
from bs4 import BeautifulSoup

for i in range(1, 11):
    res = requests.get("https://hashcode.co.kr/?page={}".format(i), user_agent)
    soup = BeautifulSoup(res.text, "html.parser")

    # 1. url 태그 모두 찾기
    # 2. 1번 안에 있는 li 태그의 text 추출
    
    ul_tags = soup.find_all("ul", "question-tags")
    for i in ul_tags:
        li_tags = ul.find_all("li")
        for li in li_tags:
            tag = li.text.strip()
            if tag not in frequency:
                frequency[tag] = 1
            else:
                frequency[tag] += 1
    time.sleep(0.5)
print(frequency)
```

가장 많이 등장한 태그를 확인한다.

```python
# Counter를 사용해 가장 빈도가 높은 value들을 추출합니다.

from collections import Counter

counter = Counter(frequency)

counter.most_common(10)
```

이를 Barplot으로 시각화한다.

```python
# Seaborn을 이용해 이를 Barplot으로 그립니다.

import seaborn as sns

x = [elem[0] for elem in counter.most_common(10)]
y = [elem[1] for elem in counter.most_common(10)]

sns.barplot(x=x, y=y)

# figure, xlabel, ylabel, title을 적절하게 설정해서 시각화를 완성해봅시다.

import matplotlib.pyplot as plt

plt.figure(figsize=(20, 10))
plt.title("Frequency of question in Hashcode")
plt.xlabel("Tag")
plt.ylabel("Frequency")

sns.barplot(x=x, y=y)

plt.show()
```

## 뭉게뭉게 단어구름, WordCloud
---
`wordcloud`는 텍스트 데이터의 빈도를 기반으로 단어 구름을 만들어주는 라이브러리이다. 한국어 문장을 다루기 위해서는 형태소 분석기가 필요하며, `konlpy`의 `Hannanum`을 사용한다.

```python
# 시각화에 쓰이는 라이브러리
import matplotlib.pyplot as plt
from wordcloud import WordCloud

# 횟수를 기반으로 딕셔너리 생성
from collections import Counter

# 문장에서 명사를 추출하는 형태소 분석 라이브러리
from konlpy.tag import Hannanum

# 워드클라우드를 만드는 데 사용할 애국가 가사입니다.

national_anthem = """
동해물과 백두산이 마르고 닳도록
하느님이 보우하사 우리나라 만세
무궁화 삼천리 화려 강산
대한 사람 대한으로 길이 보전하세
남산 위에 저 소나무 철갑을 두른 듯
바람 서리 불변함은 우리 기상일세
무궁화 삼천리 화려 강산
대한 사람 대한으로 길이 보전하세
가을 하늘 공활한데 높고 구름 없이
밝은 달은 우리 가슴 일편단심일세
무궁화 삼천리 화려 강산
대한 사람 대한으로 길이 보전하세
이 기상과 이 맘으로 충성을 다하여
괴로우나 즐거우나 나라 사랑하세
무궁화 삼천리 화려 강산
대한 사람 대한으로 길이 보전하세
"""

# Hannanum 객체를 생성한 후, .nouns()를 통해 명사를 추출합니다.

hannanum = Hannanum()
nouns = hannanum.nouns(national_anthem)
words = [noun for noun in nouns if len(noun) > 1]

words[:10]

# counter를 이용해 각 단어의 개수를 세줍니다.

counter = Counter(words)

# WordCloud를 이용해 텍스트 구름을 만들어봅시다.

wordcloud = WordCloud(
    font_path="C:\\Users\\yyt11\\EliceDigitalBaeum_Bold.ttf",
    background_color="white",
    width=1000,
    height=1000
)

img = wordcloud.generate_from_frequencies(counter)
plt.imshow(img)
```

|WordCloud|
|---|
|![WordCloud](/assets/img/contents/chart/wordcloud.png "WordCloud")|

## WordCloud로 해시코드 질문 키워드 요약
---
해시코드 질문 텍스트를 기반으로 주요 키워드를 요약할 수도 있다.

```python
# 텍스트 구름을 그리기 위해 필요한 라이브러리를 불러와봅시다.

# 시각화에 쓰이는 라이브러리
import matplotlib.pyplot as plt
from wordcloud import WordCloud

# 횟수를 기반으로 딕셔너리 생성
from collections import Counter

# 문장에서 명사를 추출하는 형태소 분석 라이브러리
from konlpy.tag import Hannanum
```

```python
# Hannanum 객체를 생성한 후, .nouns()를 통해 명사를 추출합니다.

words = []
Hannanum = Hannanum()

for question in questions:
    nouns = hannanum.nouns(question)
    words += nouns

print(len(words))
```

```python
# counter를 이용해 각 단어의 개수를 세줍니다.

counter = Counter(words)

counter
```

```python
# WordCloud를 이용해 텍스트 구름을 만들어봅시다.

wordcloud = WordCloud(
    font_path="C:\\Users\\yyt11\\EliceDigitalBaeum_Bold.ttf",
    background_color="white",
    width=1000,
    height=1000
)

img = wordcloud.generate_from_frequencies(counter)
plt.imshow(img)
plt.axis("off")
plt.show()
```