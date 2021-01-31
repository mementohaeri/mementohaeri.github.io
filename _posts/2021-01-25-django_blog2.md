---
title: "Django - Blog 앱 구축 (views, templates 페이지)"
categories :	
 - Django
tags : 
 - Django
 - DjangoBlog
 - views.py
 - templates
---

# Django Blog App 구축

## 4. View 생성
views.py 파일에 애플리케이션 로직을 위한 코드를 작성한다.
- 클래스형 뷰 (class-based view) : 코드의 재활용성을 높이고 코드의 복잡함을 막기 위해 자주 쓰는 뷰를 미리 만들어놓은 것. 예를 들어 ListView, DetailView, TemplateView, AboutView 등이 있다. 이러한 generic view를 상속받아 오버라이딩하면, django는 자동으로 그 뷰에 맞는 기능을 수행한다.

```python
from django.views.generic import ListView, DetailView
from django.views.generic.dates import ArchiveIndexView, YearArchiveView, MonthArchiveView, DayArchiveView, TodayArchiveView
from blog.models import Post

class PostLV(ListView):
    model = Post
    template_name = "blog/post_all.html"
    context_object_name = "posts" 
    paginate_by = 3     # pagination (3개씩 출력)

class PostDV(DetailView):
    model = Post

class PostAV(ArchiveIndexView):
    model = Post
    date_field = "modify_date"  # create_date는 수정 불가이기 때문

class PostYAV(YearArchiveView):
    model = Post
    make_object_list = True     # object_list를 모두 사용 가능
    date_field = "modify_date"

class PostMAV(MonthArchiveView):
    model = Post
    date_field = "modify_date"
    month_format = "%m"     # 숫자로 사용 가능

class PostDAV(DayArchiveView):
    model = Post
    date_field = "modify_date"
    month_format = "%m"

class PostTAV(TodayArchiveView):
    model = Post
    date_field = "modify_date"
    month_format = "%m"
```

1. Attribute Overriding

 - `model` : 사용할 모델을 지정한다. 함수형 뷰의 `Post.objects.all()` 로 Post 모델의  데이터를 가져오는 기능이 같다.
 - `template_name` : 사용할 템플릿 경로를 지정한다.
 - `context_object_name` : 만일 지정해주지 않는다면 template에서 `object_list` 이름으로 데이터를 가져와야 한다.
 - `date_field` : 날짜 기반 뷰에서 기준이 되는 필드를 지정한다. 해당 필드를 기준으로 Y/M/D를 검사한다. 
 - `make_object_list` : YearArchiveView 사용할 때 해당 년도에 맞는 객체의 리스트를 생성한다.

<br/>
<br/>
<br/>


## 5. Template 생성

모든 템플릿 페이지가 상속할 base.html을 생성한다. 
- base.html 페이지가 기준이 되며, 다른 template 페이지가  base.html 페이지를 오버라이딩하여 원하고자 하는 페이지를 작성할 수 있다.

```python
<!DOCTYPE html>
<html lang="ko">

<head>
<title>{% block title %}Django Web Programming{% endblock %}</title>

{% load static %}
<link rel="stylesheet" type="text/css" href="{% block stylesheet %}{% static 'css/base.css' %}{% endblock %}" />
<link rel="stylesheet" type="text/css" href="{% block extrastyle %}{% endblock %}" />

</head>

<body>

    <div id="header">
        <h2 class="maintitle">Easy&amp;Fast Django Web Programming</h2>
            <h4 class="welcome">Welcome, <a href="#">OOO</a> /
                <a href="#">Change Password</a> /
                <a href="#">Logout</a>
            </h4>
    </div>

    <div id="menu">
        <li><a href="{% url 'home' %}">Home</a></li>
        <li><a href="{% url 'bookmark:index' %}">Bookmark</a></li>
        <li><a href="{% url 'blog:index' %}">Blog</a></li>
        <li><a href="#">Photo</a></li>

        <li><a href="#">Add&bigtriangledown;</a>
            <ul>
                <li><a href="#">Bookmark</a></li>
                <li><a href="#">Blog</a></li>
                <li><a href="#">Photo</a></li>
            </ul>
        </li>
        <li><a href="#">Change&bigtriangledown;</a>
            <ul>
                <li><a href="#">Bookmark</a></li>
                <li><a href="#">Blog</a></li>
            <li><a href="#">Photo</a></li>
            </ul>
        </li>

        <li><a href="{% url 'blog:post_archive' %}">Archive</a></li>
        <li><a href="{% url 'blog:search' %}">Search</a></li>
        <li><a href="{% url 'admin:index' %}">Admin</a></li>
    </div>

    {% block content %}{% endblock %}
    {% block footer %}{% endblock %}

</body>
</html>
```

![image](https://user-images.githubusercontent.com/77096463/105660674-dc970e00-5f0e-11eb-8073-b73d7abd002e.png)

<br/>



templates/blog/post_all.html 경로에 코드를 작성한다.

```python
{% extends "base.html" %}

{% block title%} Post title {% endblock%}

{% block content %}
<div id="content">
   <h1>Blog List</h1>
   <!-- display blog list -->
   {% for post in posts %}
      <h2><a href='{{post.get_absolute_url}}'>{{post.title}}</a></h2>
      {{post.modify_date | date:"Y-m-d H:i A"}}
      <p>{{post.description}}</p>
   {% endfor%}
   <br/>

   <!-- paging link -->
   <div>
         {% if page_obj.has_previous %}
            <a href="?page={{page_obj.previous_page_number}}"> Prev </a>
         {% endif %}

         [ Page {{page_obj.number}} /  {{page_obj.paginator.num_pages}} ]
         
         {% if page_obj.has_next %}
            <a href="?page={{page_obj.next_page_number}}"> Next </a>
         {% endif %}
   </div>
</div>
{% endblock %}
```

<br/>
<br/>



출처 : https://velog.io/@hyeseong-dev/django-View-1-Gegeric-View-Overriding

