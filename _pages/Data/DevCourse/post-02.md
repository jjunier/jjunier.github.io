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

**메타 데이터 - 인코딩**
`charset`(character set)은 **문서에서 허용하는 문서의 집합**을 의미한다. charset에 선언된 문서의 집합 규칙에 따라 문서에서 사용할 수 있는 문자가 제한된다. 이에 대부분 전 세계적인 charset 집합인 `UTF-8`을 사용하는게 일반적이다.

### HTML `<STYLE>` 태그
---
문서 내용인 콘텐츠의 외형에 영향을 주는 태그들이다. 해당 태그들의 활용은 다음과 같다:

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
    </head>
    <body>              
    </body>
</html>
```