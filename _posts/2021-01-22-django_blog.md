---
title: "Django - Blog 앱 구축 (Models, Admin, urls 페이지)"
categories :	
 - Django
tags : 
 - Django
 - DjangoBlog
 - models.py
 - admin.py
 - urls.py
---

# Django App 개발 프로세스

![process](https://user-images.githubusercontent.com/77096463/105594056-668f8b80-5dd6-11eb-83ff-b5aba403b43c.png)
<br/>

1. 프로젝트 생성
   - 프로젝트 및 앱 개발에 필요한 디렉토리 및 파일 생성
2. 모델 생성
   - models.py, admin.py 
3. URLConf 생성
   - urls.py : URL-VIEW 매핑
4. View 생성
   - views.py : 애플리케이션 로직 개발 
5. Template 생성
   - 화면 UI 개발 (templates/*.html)

<br/>
<br/>

# Django Blog App 구축

## 1. 프로젝트 생성

django 프로젝트를 위한 가상환경을 생성한다. 
	- Anconda Prompt 에서 진행
```
$ conda create --name [가상환경명]
$ conda activate [가상환경명]
```
<br/>
<br/>


생성한 가상환경 내에 django 프레임워크를 설치한다.
```
$ conda install django
```
<br/>
<br/>

django 프로젝트를 생성한다.

​	- django 프로젝트 폴더 하위에 manage.py 파일이 생성되었는지 확인한다.

```
$ django-admin startproject [프로젝트명]
$ cd [프로젝트 디렉토리]
$ code .			# 현 디렉토리에서 vscode를 실행한다.
```
<br/>
<br/>


settings.py (설정파일)을 변경한다.
- TEMPLATES, DATABASES경로를 프로젝트 경로에 맞춰 변경한다.
- LANGUAGE_CODE, TIME_ZONE를 각각 한국어, 서울 시간에 맞춰준다.

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],				#  templates 경로 설정
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),				# 데이터베이스 경로 설정
    }
}

...

LANGUAGE_CODE = 'ko-kr'												

TIME_ZONE = 'Asia/Seoul'
```
<br/>
<br/>


프로젝트 관리자인 superuser를 생성한다.
- id, password, email 을 설정한다.

```
$ python manage.py createsuperuser 
```
<br/>
<br/>


startapp 명령어를 통해 프로젝트 내에 만들고자 하는 앱을 생성한다.

```
$ python manage.py startapp [앱이름]
```
<br/>
<br/>


방금 만든 앱을 settings.py 파일에 등록하여 프로젝트 에서 앱을 사용할 수 있게 수정한다.

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'bookmark.apps.BookmarkConfig',			
    'blog.apps.BlogConfig',					# blog 앱을 생성한 후 추가
]
```
<br/>
<br/>



## 2. 모델 생성

모델을 정의한 다음, models.py 파일에 코드를 추가한다.

| field       | type           | condition          | description       |
| ----------- | -------------- | ------------------ | ----------------- |
| id          | int            | PK, auto increment | 기본키            |
| title       | CharField(50)  | -                  | 포스트 제목       |
| slug        | SlugField(50)  | Unique             | 포스트 제목 별칭  |
| description | CharField(100) | Blank              | 포스트 내용 한 줄 |
| content     | TextField      | -                  | 포스트 내용       |
| create_date | DateTimeField  | auto_now_add       | 포스트 작성 날짜  |
| modify_date | DateTimeField  | auto_now           | 포스트 수정 날짜  |

<br/>

```python
from django.db import models
from django.urls import reverse

# Create your models here.
class Post(models.Model):
    title = models.CharField('TITLE', max_length=50)
    slug = models.SlugField('SLUG', unique=True, allow_unicode=True, help_text='one word for this alias')
    description = models.CharField('DESCRIPTION', max_length=100, blank=True, help_text='simple description')
    content = models.TextField('CONTENT')
    create_date = models.DateTimeField('Create Date', auto_now_add=True)
    modify_date = models.DateTimeField('Modify Date', auto_now=True)

    # title이 출력됨
    def __str__(self):
        return self.title

    # inner class - customize 하고 싶은 경우
    class Meta:
        verbose_name = 'post'
        verbose_name_plural = 'posts'
        db_table = 'blog_posts'
        # modify_date를 기준으로 descending order > 최근 데이터를 먼저 불러오기
        ordering = ('-modify_date',)
```

1. `models` : Post 클래스가 장고모델임을 명시. 장고는 이를 통해 Post 클래스가 데이터베이스에 저장되어야 한다고 인지한다.
2. Field Options
   - `allow_unicode` : 한글 지원
   - `help_text`: 해당 필드가 폼 위젯으로 표시될 때 함께 표시되는 도움말 
   - `blank` : 해당 필드 값을 입력하지 않아도 됨
   - ` unique` : 값이 True 인 경우, 해당 테이블에서 유일한 값을 가져야 함
   - `auto_now_add` :  필드 객체가 처음 생성되었을 때 현재날짜를 적용
   - `auto_now` : 필드 객체가 수정될 때마다 현재날짜로 갱신됨
3. `class Meta` : Inner class로 상위 클래스에게 메타데이터를 제공한다. 
   - `verbose_name` : 사용자가 읽기 쉬운 객체의 이름으로 관리자 화면에 표시
   - `verbose_names` : `verbose_name`과 동일하나 영어를 기준으로 복수형
   - `db_table` : 원래 장고는 <앱 + 클래스> 이름으로 테이블을 생성하는데, `db_table`을 지정하면 지정한 이름으로 테이블을 생성

<br/>
<br/>

admin 사이트에 모델을 등록한다.
​	- `python manage.py runserver` 명령어 실행 후, http://127.0.0.1:8000/admin/ 홈페이지를 통해 확인 가능

```python
from django.contrib import admin
from blog.models import Post

# Register your models here.
class PostAdmin(admin.ModelAdmin):
    list_display = ('title','modify_date')
    list_filter = ('modify_date',)	# 필터창
    search_fields = ('title','content')	# 검색창
    prepopulated_fields = {'slug':('title',)}

admin.site.register(Post, PostAdmin)
```

1. `list_display` : 객체 내용 및 연산결과 표시
2. `prepopulated_fields{'slug':('title',)}` : title 값을 기반으로 slug를 자동으로 채움 (a b -> a-b)
3. `admin.site.register(Post, PostAdmin) ` : Post 테이블을 관리자 페이지로 등록하되, PostAdmin 객체에 지정된 방식으로 관리

![image](https://user-images.githubusercontent.com/77096463/105629950-508ed300-5e89-11eb-8e37-8b87a878fc5d.png)

<br/>
<br/>

모델을 데이터베이스에 반영한다.
```
$ python manage.py makemigrations
$ python manage.py migrate
```
<br/>
<br/>


## 3. URLConf 코딩

| URL 패턴 (urls.py) | View 이름 (views.py)      | Template 파일명 (template/*.html) |
| ------------------ | ------------------------- | --------------------------------- |
| /blog/             | PostLV(ListView)          | post_all.html                     |
| /blog/post         | PostLV(ListView)          | post_all.html                     |
| /blog/post/python  | PostDV(DetailView)        | post_detail.html                  |
| /blog/archive      | PostAV(ArchiveIndexView)  | post_archive.html                 |
| /blog/2021         | PostYAV(YearArchiveView)  | post_archive_year.html            |
| /blog/2021/jan     | PostMAV(MonthArchiveView) | post_archive_month.html           |
| /blog/2021/jan/24  | PostDAV(DayArchiveView)   | post_archive_day.html             |
| /blog/today/       | PostTAV(TodayArchiveView) | post_archive_day.html             |
| /admin             | Django                    | -                                 |
<br/>
<br/>

urls.py 파일에 코드를 추가한다.
```python
from django.contrib import admin
from blog.views import *
from django.urls import path

app_name = 'blog'

urlpatterns = [
    # /blog/ : http://127.0.0.1:8000/blog/
    path('', PostLV.as_view(), name='index'),

    # /blog/post/ : http://127.0.0.1:8000/blog/post
    path('post/', PostLV.as_view(), name='post_list'),
    
    # blog/post/slug/
    path('post/<str:slug>', PostDV.as_view(), name='post_detail'),
    
    # /blog//archive/
    path('archive/', PostAV.as_view(), name='post_archive'),
    
    # /blog/2021
    path('<int:year>/', PostYAV.as_view(), name='post_year_archive'),
    
    # /2021/01/
    path('<int:year>/<str:month>', PostMAV.as_view(), name='post_month_archive'),
    
    # /2021/01/24
    path('<int:year>/<str:month>/<int:day>', PostDAV.as_view(), name='post_day_archive'),
    
    # /today/ : http://127.0.0.1:8000/blog/today
    path('today/', PostTAV.as_view(), name='post_today_archive'),

    # /search/ : http://127.0.0.1:8000/blog/search
    path('search/', SearchFormView.as_view(), name='search'),
]
```
1. `as_view()` 
   - 뷰에는 class-based view와 function-based view가 있는데, views.py 파일에서 클래스형 뷰를 사용한다.
   -  이러한 뷰는 generic을 사용하고 외부 모듈 적용이 필요한데, `as_view()`를 통해 generic view를 사용할 수 있다.
2.  path의 `name`  : path()의 네번째 파라미터로, path의 이름을 지정한다. path 명으로부터 URL 패턴 정보를 찾는 url reversing을 위한 것

<br/>
<br/>



출처 : https://tutorial.djangogirls.org/ko/django_models/ <br/>
https://has3ong.tistory.com/236 <br/>
https://m.blog.naver.com/PostView.nhn?blogId=pjok1122&logNo=221594402973&proxyReferer=https:%2F%2Fwww.google.com%2F
<br/>

