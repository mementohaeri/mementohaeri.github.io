---
title: "Django - Blog 앱 구축"
categories :	
 - Django
tags : 
 - Django
 - DjangoBlog
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
> conda create --name [가상환경명]
> conda activate [가상환경명]
```



생성한 가상환경 내에 django 프레임워크를 설치한다.

```
> conda install django
```



django 프로젝트를 생성한다.

- django 프로젝트 폴더 하위에 manage.py 파일이 생성되었는지 확인한다.

```
> django-admin startproject [프로젝트명]
> cd [프로젝트 디렉토리]
> code .			# 현 디렉토리에서 vscode를 실행한다.
```



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



프로젝트 관리자인 superuser를 생성한다.

- id, password, email 을 설정한다.

```
> python manage.py createsuperuser 
```



startapp 명령어를 통해 프로젝트 내에 만들고자 하는 앱을 생성한다.

```
> python manage.py startapp [앱이름]
```



방금 만든 앱을 settings.py 파일에 추가하여 프로젝트 에서 앱을 사용할 수 있게 수정한다.

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





