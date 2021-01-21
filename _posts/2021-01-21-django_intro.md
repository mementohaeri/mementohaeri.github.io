---
title: "Django - 개요, MTV패턴, 파일 구성"
categories :	
 - Django
tags : 
 - Django
 - MTV
---

# Django

Django (장고)는 웹 어플리케이션 개발에 사용하는 파이썬 기반의 웹 프레임워크이다. 
웹사이트를 쉽고 빠르게 개발할 수 있는 Python Full Stack Framework로 클라이언트, 서버, DB 연동을 모두 지원한다. 장고는 MTV 패턴 (Model-Template-View) 을 따른다.
<br/><br/>

# MTV 패턴

애플리케이션을 3가지의 역할로 구분한 개발 방법론인 MVC 패턴을 따르고 있으나, 컨트롤러의 기능을 django 프레임워크 자체에서 하고 있기에 보통 MTV (**M**odel - **T**emplate - **V**iew) 패턴을 따른다고 말한다.

**Model** : 테이블을 정의하여 안전하게 데이터를 저장한다.

**Template** : 사용자가 보게 될 화면의 모습을 정의한다. 

**View** : 애플리케이션의 제어 흐름 및 처리 로직을 정의하여, 처리 결과를 Template에 전달한다. (Rendering)

모델, 템플릿, 뷰 모듈 간에 독립성을 유지하기에 한 요소가 다른 요소에 영향을 주지 않도록 설계하는 방식이다.

![mtv](https://user-images.githubusercontent.com/77096463/105367713-141e6580-5c44-11eb-8c04-f1dd6294381b.png)

- ORM (Object Relational Mapping) : 데이터베이스와 객체 지향 프로그래밍 언어 간의 호환되지 않는 데이터를 변환하는 프로그래밍 기법이다. 즉, 데이터베이스는 'table'을, 객체 지향 프로그래밍 언어는 'class'를 사용하여  호환되지 않는 데이터가 발생하고 ORM은 이러한 객체와 관계형 DB 간의 데이터를 자동으로 매핑한다.

<br/>

<br/>

# Django 프로젝트 파일 구성

## models.py

테이블을 정의하는 파일이다. 
ORM (Object Relational Mapping)을 사용하여 테이블을 클래스와 매핑하여 SQL을 직접 작성하지 않아도 CRUD 기능을 클래스 객체에 대해 수행한 뒤, 데이터베이스에 반영한다. 테이블-클래스, 테이블의 컬럼-클래스 변수로 매핑하여 django.db.models.Model 클래스를 상속한다. 
models.py에서 데이터베이스 변경사항이 발생하는 경우, 실제 DB에 반영하기 위해 마이그레이션 기능을 사용한다.

<br/>

## views.py

애플리케이션의 로직을 생성하는 파일이다. 모델에서 필요한 데이터를 받아와 템플릿에 전달한다.
함수 (function-based view) 또는 클래스 (class-based view)로 생성 가능하다.

<br/>

## templates (*.html)

templates/디렉토리 하위에 페이지 별로 앱 템플릿 파일 (.html)을 저장하며, HTML 양식을 따른다.
정보를 일정한 형태로 표시하기 위한 재사용 가능한 파일이다.
<br/> 

## URLconf (urls.py)

**URLConf(configuration)** : URL과 일치하는 view를 찾기 위한 패턴들의 집합

페이지 요청 시 가장 먼저 호출되며 URL-View(함수 or 메서드)를 매핑한다.
프로젝트 URL과 앱 URL로 분리하여 각각 구성하는 방법이 권장된다.
django 서버로 http 요청이 들어오면, URLConf의 매핑 리스트를 순차적으로 찾아 검색한다.
<br/>

## settings.py

데이터베이스, 템플릿 항목, 정적 파일 항목, 애플리케이션 등록, 타임존 등을 설정한다.
django에서는 default 데이터베이스로 Sqlite3를 사용한다. 

<br/>


## admin.py

테이블 내용을 열람하고 수정하는 기능을 제공하며, 사용자가 비즈니스 로직 개발에 필요한 테이블을 관리할 수 있는 기능을 제공한다. User 및 Group 테이블을 관리하여 settings.py에 django.contrib.auth 애플리케이션이 등록된다.

<br/>

## 개발용 웹 서버

django에서는 테스트용 runserver가 제공된다.
상용화를 위해서는 Apache, Nginx 등의 상용 서버로 변경해야 한다. 
<br/>

