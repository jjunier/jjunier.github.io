---
title: "[6기] 데브코스 DE WIL 03 | Django API 서버"
tags:
    - DevCourse
    - Data Engineering
    - Server
date: "2025-04-18"
thumbnail: "/assets/img/thumbnail/devcourse.png"
bookmark: true
---

## 이번 주 학습 목표
---
- Django 프로젝트·앱 생성부터 웹 앱 → API 서버로 확장되는 **전체 개발 흐름을 직접 구현**하며, **Django 프레임워크의 구조와 역할을 입체적으로 이해**한다.
- **URL → View → ORM → Serializer → Response(JSON)** 로 이어지는 요청 처리 과정을 실습을 통해 체득하고, **함수 기반 뷰부터 Generic API View까지 여러 추상화 단계의 차이와 목적을 이해**한다.
- Django ORM, Admin, Shell, DRF를 활용해 CRUD API를 완성하고, User·인증·권한(Owner 기반 접근 제어)까지 포함한 **실무 수준의 데이터 관리 및 접근 제어 구조를 구현**한다.

## Django Project 생성하기
---
Django 개발의 시작은 하나의 `프로젝트(Project)`를 생성하는 것이다. 여기서 프로젝트란 **전체 설정과 앱들을 감싸는 최상위 단위 역할**을 한다.

```shell
$ django-admin startproject mysite
```

프로젝트가 생성되면, 기본 설정이 포함된 디렉터리 구조가 함께 만들어진다. 이후 개발 서버를 실행해 정상적으로 동작하는지 확인한다.

```shell
$ python manage.py runserver
```

## Django App 생성하기
---
**Django 프로젝트 내부에서는 실제 기능 단위**를 `앱(App)`이라는 개념으로 분리한다. 예제로 `polls`라는 앱을 생성해본다.

```shell
$ python manage.py startapp polls
```

앱 내부의 `view.py`에 가장 단순한 응답을 반환하는 함수를 작성한다.

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello World.")
```

## URL 연결하기
---
작성한 view 함수가 실제 URL 요청과 연결되기 위해서는 URL 설정이 필요하다. 프로젝트 단위의 urls.py에서 앱의 URL을 포함시킨다.

```python
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path("admin/", admin.site.urls),
    path("polls/", include("polls.urls")),
]
```

이후 앱 내부에 `urls.py`를 생성하고 view와 연결한다.

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

## URL 경로(Path) 확장하기
---
하나의 앱 안에서도 여러 URL 경로를 가질 수 있다.
view 함수와 URL을 추가로 연결해본다.

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")

def some_url(request):
    return HttpResponse("Some url을 구현해 봤습니다.")

from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
    path("some_url", views.some_url),
]
```

## 모델(Model) 만들기
---
Django의 핵심 개념 중 하나는 **ORM(Object Relational Mapping)**이다.
모델을 통해 데이터베이스 구조를 코드로 정의한다.

먼저 앱을 프로젝트 설정에 등록한다.

```python
# mysite/settings.py

INSTALLED_APPS = [
    ...
    "polls.apps.PollsConfig",
]
```

이후 models.py에 모델을 정의한다.

```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

## 마이그레이션(Migration)
---
모델 변경 사항을 데이터베이스에 반영하기 위해 마이그레이션을 생성하고 실행한다.

```shell
$ python manage.py makemigrations polls
```

생성될 SQL을 확인할 수도 있다.

```shell
$ python manage.py sqlmigrate polls 0001
```

마이그레이션을 실제로 적용한다.

```shell
$ python manage.py migrate
```

## 다양한 모델 필드 활용
---
Django는 다양한 필드 타입을 기본으로 제공한다.

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")
    # is_something = models.BooleanField(default=False)
    # average_score = models.FloatField(default=0.0)
```

기본 데이터베이스로는 SQLite가 사용된다.

```shell
$ sqlite3 db.sqlite3
```

마이그레이션을 이전 상태로 되돌릴 수도 있다.

```shell
$ python manage.py migrate polls 0001
```

## Django Admin – 관리자 계정 생성
---
Django는 기본적으로 관리자 페이지를 제공한다. 관리자 계정을 생성한다.

```shell
$ python manage.py createsuperuser
```

### Admin에 모델 등록하기
---
관리자 페이지에서 모델을 관리하려면 등록이 필요하다.

```python
# polls/admin.py

from django.contrib import admin
from .models import *

admin.site.register(Question)
admin.site.register(Choice)
```

문자열 표현을 추가하면 관리자 화면에서 더 읽기 쉬워진다.

```python
def __str__(self):
    return f"제목:{self.question_text}, 날짜:{self.pub_date}"
```

## Django Shell 사용하기
---
Django Shell은 ORM을 직접 다뤄볼 수 있는 실습 도구다.

```shell
$ python manage.py shell

>>> from polls.models import *
>>> Question.objects.all()
>>> Choice.objects.all()
```

관계형 데이터 접근도 가능하다.

```shell
>>> choice.question
>>> question.choice_set.all()
```

### 현재 시간 다루기
---

```shell
>>> from datetime import datetime
>>> datetime.now()

>>> from django.utils import timezone
>>> timezone.now()
```

### 레코드 생성하기
---

```shell
>>> q1 = Question(question_text="커피 vs 녹차")
>>> q1.pub_date = timezone.now()
>>> q1.save()

>>> q3.choice_set.create(choice_text="b")
```

### 레코드 수정 및 삭제
---

```shell
>>> q = Question.objects.last()
>>> q.question_text += "???"
>>> q.save()

>>> choice.delete()
```

## Django Shell – 모델 필터링(Model Filtering)
---
Django ORM은 모델 객체를 SQL처럼 조회할 수 있도록 get()과 filter()를 제공한다. get()은 하나의 결과만 기대할 때 사용하며, 조건에 맞는 데이터가 여러 개면 예외가 발생한다.

```shell
>>> from polls.models import *

>>> Question.objects.get(id=1)
>>> q = Question.objects.get(question_text__startswith='휴가를')
>>> Question.objects.get(pub_date__year=2023)
polls.models.Question.MultipleObjectsReturned: get() returned more than one Question
```

반대로 filter()는 조건에 맞는 결과를 QuerySet(목록) 형태로 반환한다.
여기서 .count()로 개수를 확인할 수도 있다.

```shell
>>> Question.objects.filter(pub_date__year=2023)
>>> Question.objects.filter(pub_date__year=2023).count()
```

또 하나 유용한 점은, ORM이 실제로 어떤 SQL을 만드는지 확인할 수 있다는 것이다.

```shell
>>> print(Question.objects.filter(pub_date__year=2023).query)
>>> print(Question.objects.filter(question_text__startswith='휴가를').query)
```

관계형 조회도 자연스럽게 이어진다. ForeignKey 관계로 연결된 데이터를 .choice_set으로 접근할 수 있다.

```shell
>>> q = Question.objects.get(pk=1)
>>> q.choice_set.all()
>>> print(q.choice_set.all().query)
```

### 모델 필터링(Model Filtering) 2
---
ORM 필터링은 다양한 “룩업(lookup)” 연산자를 통해 확장된다. startswith, contains, gt(>) 같은 연산자를 활용하면 조건을 세밀하게 만들 수 있다.

```shell
>>> from polls.models import *

>>> Question.objects.filter(question_text__startswith='휴가를')
>>> Question.objects.filter(question_text__contains='휴가')

>>> Choice.objects.filter(votes__gt=0)
>>> print(Choice.objects.filter(votes__gt=0).query)
```

데이터를 바꿔보고 저장하면서 ORM 흐름을 손에 익히는 것도 중요하다.

```shell
>>> choice = Choice.objects.first()
>>> choice.votes = 5
>>> choice.save()
```

또한 정규표현식 기반 필터링도 가능하다.

```shell
>>> print(Question.objects.filter(question_text__regex=r'^휴가.*어디').query)
```

###  Django 모델 관계 기반 필터링
---
관계형 모델을 연결해두면, ORM은 "JOIN을 직접 쓰지 않아도" 관계를 타고 들어가 필터링을 할 수 있다. 이를 위해 __(double underscore)로 필드를 이어붙인다.

먼저 모델의 출력 형태를 보기 좋게 정리한다.

```python
# polls/models.py
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return f'제목:{self.question_text}, 날짜:{self.pub_date}'

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return f'[{self.question.question_text}]{self.choice_text}'
```

이제 Choice를 조회하면서 Question의 필드 조건으로 필터링할 수 있다.

```shell
>>> from polls.models import *
>>> Choice.objects.filter(question__question_text__startswith='휴가')
```

반대로 특정 조건을 제외하고 가져오고 싶다면 exclude()를 사용한다.

```shell
>>> Question.objects.exclude(question_text__startswith='휴가')
```

### Django Shell – 모델 메소드
---
모델은 단순히 DB 스키마만 정의하는 곳이 아니라, "해당 데이터가 가져야 할 행동"을 메소드로 담을 수 있다. 예를 들어, "최근 게시물인지" 같은 판단 로직을 모델에 넣어두면 재사용이 쉬워진다.

```python
# polls/models.py
from django.utils import timezone
import datetime
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    def __str__(self):
        new_badge = 'NEW!!!' if self.was_published_recently() else ''
        return f'{new_badge} 제목:{self.question_text}, 날짜:{self.pub_date}'
```

## 뷰(Views)와 템플릿(Templates)
---
이제부터는 Shell에서 조회하던 데이터를 실제 웹 화면으로 보여준다. View에서는 데이터를 조회하고, Template로 전달할 context를 구성해 렌더링한다.

```python
# polls/views.py
from .models import *
from django.shortcuts import render

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'first_question': latest_question_list[0]}
    return render(request, 'polls/index.html', context)
```

정렬/슬라이싱이 어떤 SQL이 되는지도 확인할 수 있다.

```shell
>>> from polls.models import *
>>> print(Question.objects.order_by('-pub_date')[:5].query)
```

템플릿에서는 전달된 변수를 출력한다.

```html
<!-- polls/templates/polls/index.html -->
{% raw %}
<ul>
  <li>{{ first_question }}</li>
</ul>
{% endraw %}
```

## 템플릿에서 제어문 사용하기
---
템플릿은 단순 출력뿐 아니라, 조건문/반복문을 제공한다. 리스트를 전달하고 반복 렌더링하도록 수정한다.

```python
# polls/views.py
from .models import *
from django.shortcuts import render

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'questions': latest_question_list}
    # context = {'questions': []}
    return render(request, 'polls/index.html', context)
```

```html
{% raw %}
{% if questions %}
<ul>
  {% for question in questions %}
    <li>{{ question }}</li>
  {% endfor %}
</ul>
{% else %}
<p>no questions</p>
{% endif %}
{% endraw %}
```

## 상세(detail) 페이지 만들기
---
목록만 보여주는 것에서 끝나지 않고, 특정 질문의 상세 페이지로 이동할 수 있어야 한다. URL에서 question_id를 받아 해당 객체를 조회해 템플릿에 전달한다.

```python
# polls/views.py
def detail(request, question_id):
    question = Question.objects.get(pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

URL 패턴도 정수 파라미터를 받도록 추가한다.

```python
# polls/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('some_url', views.some_url),
    path('<int:question_id>/', views.detail, name='detail'),
]
```

템플릿에서는 연결된 Choice들을 순회해 출력한다.

```html
{% raw %}
<h1>{{ question.question_text }}</h1>
<ul>
  {% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
  {% endfor %}
</ul>
{% endraw %}
```

### 상세 페이지로 링크 추가하기
---
이제 목록 페이지에서 각 질문을 클릭하면 상세 페이지로 이동하도록 링크를 연결한다. 이를 위해 app_name을 지정하고 namespaced URL을 사용한다.

```python
# polls/urls.py
from django.urls import path
from . import views

app_name = 'polls'

urlpatterns = [
    path('', views.index, name='index'),
    path('some_url', views.some_url),
    path('<int:question_id>/', views.detail, name='detail'),
]
```

```html
{% raw %}
{% if questions %}
<ul>
  {% for question in questions %}
    <li>
      <a href="{% url 'polls:detail' question.id %}">
        {{ question.question_text }}
      </a>
    </li>
  {% endfor %}
</ul>
{% else %}
<p>no questions</p>
{% endif %}
{% endraw %}
```

### 404 에러 처리하기
---
get()으로 객체를 가져올 때 데이터가 없으면 예외가 발생한다. Django에서는 이를 깔끔하게 처리하기 위해 get_object_or_404()를 제공한다.

```python
from django.shortcuts import render, get_object_or_404

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

## 폼(Forms) – 투표 기능 만들기
---
이제 상세 페이지에서 사용자가 선택지를 고르고 제출하면, 서버가 이를 받아 votes를 증가시키는 흐름을 만든다. 폼은 POST 요청으로 서버에 데이터를 전달한다.

```python
from django.http import HttpResponseRedirect
from django.urls import reverse
from django.shortcuts import get_object_or_404, render

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': '선택이 없습니다.'
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:index'))
```

URL도 vote 엔드포인트를 추가한다.

```python
# polls/urls.py
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

템플릿에서 라디오 버튼과 csrf 토큰을 포함한 폼을 만든다.

```html
{% raw %}
<form action="{% url 'polls:vote' question.id %}" method="post">
  {% csrf_token %}
  <h1>{{ question.question_text }}</h1>

  {% if error_message %}
    <p><strong>{{ error_message }}</strong></p>
  {% endif %}

  {% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label>
    <br>
  {% endfor %}

  <input type="submit" value="Vote">
</form>
{% endraw %}
```

## 에러 방어하기 1
---
사용자가 선택 없이 제출했거나, 잘못된 ID로 요청했을 때를 대비해 에러 메시지를 더 구체화한다.

```python
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': f"선택이 없습니다. id={request.POST['choice']}"
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

## 에러 방어하기 2 – 동시성(F) 처리
---
단순히 votes += 1은 동시 요청이 들어올 경우 값이 꼬일 수 있다. 이를 방지하기 위해 DB 레벨에서 안전하게 증가시키는 F() 표현식을 사용한다.

```python
from django.db.models import F
from django.urls import reverse

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': f"선택이 없습니다. id={request.POST['choice']}"
        })
    else:
        selected_choice.votes = F('votes') + 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:index'))
```

## 결과(result) 조회 페이지
---
투표가 끝난 뒤 결과를 확인할 수 있도록 결과 페이지를 만든다. vote 이후 redirect를 result로 보내고, result view에서 해당 질문과 선택지 정보를 렌더링한다.

```python
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.db.models import F
from django.http import HttpResponseRedirect

def vote(request, question_id):
    ...
    else:
        selected_choice.votes = F('votes') + 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:result', args=(question.id,)))

def result(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/result.html', {'question': question})
```

```html
{% raw %}
<h1>{{ question.question_text }}</h1><br>
{% for choice in question.choice_set.all %}
  <label>
    {{ choice.choice_text }} -- {{ choice.votes }}
  </label>
  <br>
{% endfor %}
{% endraw %}
```

URL도 result 엔드포인트를 추가한다.

```python
# polls/urls.py
from django.urls import path
from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
    path('<int:question_id>/result/', views.result, name='result'),
]
```

### Django Admin 편집 페이지 커스터마이징
---
Django Admin은 기본 설정만으로도 강력하지만, 실제 운영 관점에서는 "입력 폼"을 더 읽기 쉽게 구성하는 일이 중요하다. 특히, Question과 Choice처럼 부모-자식 관계가 있는 모델에서는, 한 화면에서 함께 편집할 수 있도록 만들면 관리 효율이 크게 올라간다.

아래 설정은 Question 편집 페이지에서 Choice를 인라인(inline) 형태로 함께 수정할 수 있도록 만든다. 또한 입력 폼을 섹션 단위로 나누고, 특정 필드를 읽기 전용으로 설정한다.

```python
# polls/admin.py
from django.contrib import admin
from .models import Choice, Question

admin.site.register(Choice)

class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('질문 섹션', {'fields': ['question_text']}),
        ('생성일', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    readonly_fields = ['pub_date']
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```

- `TabularInline` : 자식 모델을 테이블 형태로 나열해 입력할 수 있게 만든다.
- `extra` : 기본으로 몇 개의 빈 입력 폼을 더 보여줄지 설정한다.
- `fieldsets` : 편집 폼을 섹션별로 나눠 가독성을 높인다.
- `readonly_fields` : 생성일처럼 수정되면 안 되는 값을 읽기 전용으로 만든다.

### Django Admin 목록 페이지 커스터마이징
---
관리자 페이지에서 “편집”만큼 중요한 것이 "목록 화면"이다. 데이터가 쌓이면 목록에서 빠르게 탐색하고 필터링할 수 있어야 하며, 이때 list_filter, search_fields 같은 옵션이 효과적이다.

먼저,  모델에서 관리자 목록에 표시할 수 있는 "계산 컬럼"을 추가한다. 여기서는 하루 기준으로 최근 생성 여부를 boolean으로 보여준다.

```python
# polls/models.py
import datetime
from django.db import models
from django.utils import timezone
from django.contrib import admin

class Question(models.Model):
    question_text = models.CharField(max_length=200, verbose='질문')
    pub_date = models.DateTimeField(auto_now_add=True, verbose='생성일')

    @admin.display(boolean=True, description='최근생성(하루기준)')
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    def __str__(self):
        return f'제목: {self.question_text}, 날짜: {self.pub_date}'
```

그리고 Admin에서 필터/검색 설정을 추가한다.

```python
# polls/admin.py
from django.contrib import admin
from .models import Choice, Question

admin.site.register(Choice)

class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('질문 섹션', {'fields': ['question_text']}),
        ('생성일', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    readonly_fields = ['pub_date']
    inlines = [ChoiceInline]
    list_filter = ['pub_date']
    search_fields = ['question_text', 'choice__choice_text']

admin.site.register(Question, QuestionAdmin)
```

- `list_filter` : 생성일 기준 필터 UI가 자동으로 생긴다.
- `search_fields` : 텍스트 검색이 가능해진다.

관계 기반 검색(choice__choice_text)처럼 “모델 관계를 타고” 검색 범위를 확장할 수 있다.

### Serializer
---
이제부터는 "화면 렌더링" 중심이었던 흐름에서, REST API 형태로 데이터를 주고받는 구조로 넘어간다.
Django REST Framework(DRF)에서 Serializer는 모델 객체를 JSON으로 변환(Serialize)하고, JSON을 검증해 모델로 복원(Deserialize)하는 역할을 한다.

아래는 Serializer 클래스를 직접 정의한 방식이다.

```python
# polls_api/serializers.py
from rest_framework import serializers
from polls.models import Question

class QuestionSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    question_text = serializers.CharField(max_length=200)
    pub_date = serializers.DateTimeField(read_only=True)

    def create(self, validated_data):
        return Question.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.question_text = validated_data.get('question_text', instance.question_text)
        instance.save()
        return instance
```

- read_only=True : 클라이언트 입력값으로 받지 않고 응답에만 포함한다.
- create() / update() : .save() 호출 시 실제 DB 반영 로직이 여기서 실행된다.

### Django Shell에서 Serializer 사용하기
---
Serializer는 "HTTP 요청이 오기 전에도" Shell에서 충분히 연습할 수 있다. 이 실습에서는 **Serialize → JSON 렌더링 → Deserialize → 검증 → 저장(Create/Update) 흐름**을 한 번에 경험한다.

```python
# polls_api/serializers.py
from rest_framework import serializers
from polls.models import Question

class QuestionSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    question_text = serializers.CharField(max_length=200)
    pub_date = serializers.DateTimeField(read_only=True)

    def create(self, validated_data):
        return Question.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.question_text = validated_data.get('question_text', instance.question_text) + '[시리얼라이저에서 업데이트]'
        instance.save()
        return instance
```

```shell
# Serialize
>>> from polls.models import Question
>>> from polls_api.serializers import QuestionSerializer
>>> q = Question.objects.first()
>>> serializer = QuestionSerializer(q)
>>> serializer.data

>>> from rest_framework.renderers import JSONRenderer
>>> json_str = JSONRenderer().render(serializer.data)

# Deserialize
>>> import json
>>> data = json.loads(json_str)
>>> serializer = QuestionSerializer(data=data)
>>> serializer.is_valid()
>>> serializer.validated_data
>>> new_question = serializer.save()  # Create

# Update
>>> data = {'question_text': '제목수정'}
>>> serializer = QuestionSerializer(new_question, data=data)
>>> serializer.is_valid()
>>> serializer.save()

# Validation 실패 케이스
>>> long_text = "abcd"*300
>>> serializer = QuestionSerializer(data={'question_text': long_text})
>>> serializer.is_valid()
>>> serializer.errors
```

여기서 핵심은, .is_valid()가 통과해야 .save()가 가능하고, 검증 실패 시 errors로 어떤 제약에 걸렸는지 확인할 수 있다는 점이다.

### ModelSerializer
---
직접 필드를 일일이 선언하는 대신, 모델 정보를 기반으로 Serializer를 자동 생성할 수도 있다. ModelSerializer는 실무에서 훨씬 자주 사용되는 방식이다.

```python
# polls_api/serializers.py
from rest_framework import serializers
from polls.models import Question

class QuestionSerializer(serializers.ModelSerializer):
    class Meta:
        model = Question
        fields = ['id', 'question_text', 'pub_date']
```

Shell에서 구조를 출력해보면 자동으로 필드가 구성된 것을 확인할 수 있다.

```shell
>>> from polls_api.serializers import QuestionSerializer
>>> print(QuestionSerializer())
>>> serializer = QuestionSerializer(data={'question_text':'모델시리얼라이저로 만들어 봅니다.'})
>>> serializer.is_valid()
>>> serializer.save()
```

## GET – 질문 목록 조회 API 만들기
---
이제 Serializer를 실제 API 응답에 연결한다. 가장 먼저 구현하기 좋은 API는 "목록 조회(GET)"이다.

```python
# polls_api/views.py
from polls.models import Question
from polls_api.serializers import QuestionSerializer
from rest_framework.response import Response
from rest_framework.decorators import api_view

@api_view()
def question_list(request):
    questions = Question.objects.all()
    serializer = QuestionSerializer(questions, many=True)
    return Response(serializer.data)
```

URL을 연결한다.

```python
# polls_api/urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('question/', question_list, name='question-list')
]
```

프로젝트 URL에도 include로 연결한다.

```python
# mysite/urls.py
from django.urls import include, path
from django.contrib import admin

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
    path('rest/', include('polls_api.urls')),
]
```

## HTTP Methods와 CRUD
---
REST API를 구현할 때 CRUD는 보통 다음 HTTP 메서드에 매핑된다.

- Create : POST
- Read : GET
- Update : PUT
- Delete : DELETE

즉, 같은 URL이라도 메서드가 달라지면 서버의 동작이 달라진다.

### POST – 질문 생성 API 추가하기
---
기존 question_list 뷰에 POST 처리까지 추가해 "조회 + 생성"을 한 엔드포인트로 만든다.

```python
# polls_api/views.py
from rest_framework.decorators import api_view
from polls.models import Question
from polls_api.serializers import QuestionSerializer
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET','POST'])
def question_list(request):
    if request.method == 'GET':
        questions = Question.objects.all()
        serializer = QuestionSerializer(questions, many=True)
        return Response(serializer.data)

    if request.method == 'POST':
        serializer = QuestionSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### PUT / DELETE – 상세 API 만들기
---
목록 엔드포인트와 별개로, 특정 리소스 1개를 다루는 상세 엔드포인트를 만든다. 여기서는 /question/`<id>`/ 형태로 하나의 Question을 조회/수정/삭제한다.

```python
# polls_api/views.py
from django.shortcuts import get_object_or_404

@api_view(['GET', 'PUT', 'DELETE'])
def question_detail(request, id):
    question = get_object_or_404(Question, pk=id)

    if request.method == 'GET':
        serializer = QuestionSerializer(question)
        return Response(serializer.data)

    if request.method == 'PUT':
        serializer = QuestionSerializer(question, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_200_OK)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    if request.method == 'DELETE':
        question.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

```python
# polls_api/urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('question/', question_list, name='question-list'),
    path('question/<int:id>/', question_detail, name='question-detail'),
]
```

### Class 기반의 뷰(Views)로 바꾸기
---
함수 기반 뷰(FBV)로도 충분하지만, API가 늘어나면 구조화가 필요해진다. DRF는 APIView를 통해 클래스 기반 구성도 지원한다.

```python
# polls_api/views.py
from rest_framework.views import APIView

class QuestionList(APIView):
    def get(self, request):
        questions = Question.objects.all()
        serializer = QuestionSerializer(questions, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = QuestionSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class QuestionDetail(APIView):
    def get(self, request, id):
        question = get_object_or_404(Question, pk=id)
        serializer = QuestionSerializer(question)
        return Response(serializer.data)

    def put(self, request, id):
        question = get_object_or_404(Question, pk=id)
        serializer = QuestionSerializer(question, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_200_OK)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, id):
        question = get_object_or_404(Question, pk=id)
        question.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

```python
# polls_api/urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('question/', QuestionList.as_view(), name='question-list'),
    path('question/<int:id>/', QuestionDetail.as_view(), name='question-detail'),
]
```

### Mixin으로 CRUD 조립하기
---
API 패턴이 반복되면, DRF에서 제공하는 Mixin을 이용해 코드를 더 줄일 수 있다. 핵심은 "queryset과 serializer_class만 지정하면, CRUD 동작을 조합할 수 있다"는 점이다.

```python
# polls_api/views.py
from polls.models import Question
from polls_api.serializers import QuestionSerializer
from rest_framework import mixins, generics

class QuestionList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

class QuestionDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

```python
# polls_api/urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('question/', QuestionList.as_view(), name='question-list'),
    path('question/<int:pk>/', QuestionDetail.as_view(), name='question-detail'),
]
```

### Generic API View로 더 단순하게
---
Mixin까지 익숙해지면, DRF의 제네릭 뷰는 더 간결한 형태를 제공한다. List+Create, Retrieve+Update+Destroy 조합은 실무에서도 가장 흔한 기본 세트다.

```python
# polls_api/views.py
from polls.models import Question
from polls_api.serializers import QuestionSerializer
from rest_framework import generics

class QuestionList(generics.ListCreateAPIView):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer

class QuestionDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer
```

## User 추가하기
---
지금까지의 Question/Choice 모델은 "누가 만들었는지"에 대한 정보가 없었다. API 서버 관점에서는 리소스의 소유자(Owner)가 있어야 인증/권한을 적용할 수 있다. 이를 위해 Question 모델에 auth.User와의 관계를 추가한다.

```python
# polls/models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200, verbose_name='질문')
    pub_date = models.DateTimeField(auto_now_add=True, verbose_name='생성일')
    owner = models.ForeignKey(
        'auth.User',
        related_name='questions',
        on_delete=models.CASCADE,
        null=True
    )

    @admin.display(boolean=True, description='최근생성(하루기준)')
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    def __str__(self):
        return f'제목:{self.question_text}, 날짜:{self.pub_date}'
```

여기서 핵심은 related_name='questions'다. 이 설정 덕분에 User 객체에서 user.questions.all()처럼 "역참조"가 가능해진다.

Shell에서 실제로 연결이 잘 되었는지 확인한다.

```shell
>>> from django.contrib.auth.models import User
>>> User.objects.all()

>>> from polls.models import *
>>> user = User.objects.first()
>>> user.questions.all()
>>> print(user.questions.all().query)
SELECT "polls_question"."id", "polls_question"."question_text", "polls_question"."pub_date", "polls_question"."owner_id"
FROM "polls_question"
WHERE "polls_question"."owner_id" = 1
```

### User 관리하기
---
User를 단순히 “DB에 존재한다”에서 끝내지 않고, REST API로 조회할 수 있도록 구성한다. 먼저, UserSerializer를 만들고, User가 만든 Question들을 함께 노출한다.

```python
# polls_api/serializers.py
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    questions = serializers.PrimaryKeyRelatedField(
        many=True,
        queryset=Question.objects.all()
    )

    class Meta:
        model = User
        fields = ['id', 'username', 'questions']
```

그리고 Generic API View로 User 목록/상세 조회 엔드포인트를 만든다.

```python
# polls_api/views.py
from django.contrib.auth.models import User
from polls_api.serializers import UserSerializer

class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer

class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

마지막으로 urls에 연결해 API 엔드포인트로 노출한다.

```python
# polls_api/urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('question/', QuestionList.as_view(), name='question-list'),
    path('question/<int:pk>/', QuestionDetail.as_view()),
    path('users/', UserList.as_view(), name='user-list'),
    path('users/<int:pk>/', UserDetail.as_view()),
]
```

## Form을 사용하여 User 생성하기
---
이번에는 API가 아니라, Django가 제공하는 “웹 폼 기반 회원가입” 흐름을 만든다. Django에는 기본 회원가입 폼인 UserCreationForm이 제공되며, 이를 CreateView로 감싸면 회원가입 화면을 쉽게 구성할 수 있다.

```python
# polls/views.py
from django.views import generic
from django.urls import reverse_lazy
from django.contrib.auth.forms import UserCreationForm

class SignupView(generic.CreateView):
    form_class = UserCreationForm
    success_url = reverse_lazy('user-list')
    template_name = 'registration/signup.html'
```

템플릿에서는 폼을 렌더링하고 POST로 제출한다.

```html
{% raw %}
<!-- polls/templates/registration/signup.html -->
<h2>회원가입</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">가입하기</button>
</form>
{% endraw %}
```

URL에 회원가입 페이지를 연결한다.

```python
# polls/urls.py
from django.urls import path
from . import views
from .views import *

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
    path('<int:question_id>/result/', views.result, name='result'),
    path('signup/', SignupView.as_view()),
]
```

reverse_lazy('user-list')가 실제로 어떤 URL을 가리키는지도 Shell에서 확인할 수 있다.

```shell
>>> from django.urls import reverse_lazy
>>> reverse_lazy('user-list')
'/rest/users/'
```

### Serializer를 사용하여 User 생성하기
---
웹 폼 방식이 “브라우저 화면 중심”이라면, API 서버에서는 “JSON 기반 회원가입”이 필요하다. 이를 위해 RegisterSerializer를 만들어 password 검증 + password 해싱 저장까지 처리한다.

```python
# polls_api/serializers.py
class RegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(
        write_only=True,
        required=True,
        validators=[validate_password]
    )
    password2 = serializers.CharField(write_only=True, required=True)

    def validate(self, attrs):
        if attrs['password'] != attrs['password2']:
            raise serializers.ValidationError({"password": "두 패스워드가 일치하지 않습니다."})
        return attrs

    def create(self, validated_data):
        user = User.objects.create(username=validated_data['username'])
        user.set_password(validated_data['password'])
        user.save()
        return user

    class Meta:
        model = User
        fields = ['username', 'password', 'password2']
```

View는 CreateAPIView로 간단히 구성한다.

```python
# polls_api/views.py
from polls_api.serializers import RegisterSerializer

class RegisterUser(generics.CreateAPIView):
    serializer_class = RegisterSerializer
```

URL에 회원가입 API 엔드포인트를 연결한다.

```python
# polls_api/urls.py
from django.urls import path
from .views import *

urlpatterns = [
    path('question/', QuestionList.as_view(), name='question-list'),
    path('question/<int:pk>/', QuestionDetail.as_view()),
    path('users/', UserList.as_view(), name='user-list'),
    path('users/<int:pk>/', UserDetail.as_view()),
    path('register/', RegisterUser.as_view()),
]
```

## User 권한 관리
---
User가 생겼다면, 이제 API에서 “누구나 수정/삭제 가능한 상태”를 끝내고 권한을 적용해야 한다. 먼저 DRF가 제공하는 로그인/로그아웃 UI를 사용하기 위해 api-auth/를 연결한다.

```python
# polls_api/urls.py
from django.urls import path, include
from .views import *

urlpatterns = [
    path('question/', QuestionList.as_view(), name='question-list'),
    path('question/<int:pk>/', QuestionDetail.as_view()),
    path('users/', UserList.as_view(), name='user-list'),
    path('users/<int:pk>/', UserDetail.as_view()),
    path('register/', RegisterUser.as_view()),
    path('api-auth/', include('rest_framework.urls')),
]
```

로그인/로그아웃 이후 이동할 URL도 설정한다.

```python
# mysite/settings.py
from django.urls import reverse_lazy

LOGIN_REDIRECT_URL = reverse_lazy('question-list')
LOGOUT_REDIRECT_URL = reverse_lazy('question-list')
```

QuestionSerializer에는 owner를 읽기 전용으로 노출한다.
이렇게 하면 응답에서는 owner가 보이되, 클라이언트가 임의로 owner를 바꾸는 것은 막을 수 있다.

```python
# polls_api/serializers.py
class QuestionSerializer(serializers.ModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')

    class Meta:
        model = Question
        fields = ['id', 'question_text', 'pub_date', 'owner']
```

다음으로 “소유자만 수정/삭제 가능” 정책을 커스텀 Permission으로 만든다.

```python
# polls_api/permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.owner == request.user
```

마지막으로 View에 권한을 적용한다.

```python
# polls_api/views.py
from rest_framework import generics, permissions
from .permissions import IsOwnerOrReadOnly

class QuestionList(generics.ListCreateAPIView):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)

class QuestionDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Question.objects.all()
    serializer_class = QuestionSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly]
```

- 목록 조회(GET)는 누구나 가능하다.
- 생성(POST)은 로그인 사용자만 가능하다.
- 수정/삭제(PUT/DELETE)는 로그인 + “소유자”만 가능하다.
- perform_create()에서 owner를 서버가 강제로 주입하여 “조작 불가”하게 만든다.