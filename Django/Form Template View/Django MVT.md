# Django MVT

---
title: Django MVT
date: 2017-12-29373147 15:12:57
---

# Django MTV

> 内容来源：[Python and Django Full Stack Web Developer Bootcamp](https://www.udemy.com/python-and-django-full-stack-web-developer-bootcamp/learn/v4/overview)

> 翻译整理：CK

## 1. 模型及数据库

### 说明：

* We use Models to incorporate Database into a Django Project
* Django comes equipped with SQLite
* Django can connect to a variety of SQL engine backends
* In the setting.py file we can edit the ENGIN parameter used for DATABASE
* To create an actual model, we use a class structure inside the relevant applications models.py file
* Each attribute of the class represents a field, which is like a column name with constrains in SQL
* Each attribute has type of field and constraints.
* Models will reference each other to represent the table relationships.
* A foreign key denotes that the column coincides with a primary key of another table

### 步骤:


### 1. 在setting.py中注册 app: first\_app

```
INSTALLED_APPS = [
'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',
'first_app',
]
```

### 2. 添加model（数据表），atrribute（字段），字段类型，约束及外键
_Edit first_app/models.py_

```py
from django.db import models


# Create your models here.


class Topic(models.Model):
top_name = models.CharField(max_length=246, unique=True)

def __str__(self):
return self.top_name


class Webpage(models.Model):
topic = models.ForeignKey(Topic, on_delete=models.CASCADE)
name = models.CharField(max_length=264, unique=True)
url = models.URLField(unique=True)

def __str__(self):
return self.name


class AccessRecord(models.Model):
name = models.ForeignKey(Webpage,on_delete=models.CASCADE)
date = models.DateField(unique=True)

def __str__(self):
return str(self.date)
```

### 3. 移植

```
root@38c80f8e29c3:/code# python manage.py migrate
Operations to perform:
Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
Applying contenttypes.0001_initial... OK
Applying auth.0001_initial... OK
Applying admin.0001_initial... OK
Applying admin.0002_logentry_remove_auto_add... OK
Applying contenttypes.0002_remove_content_type_name... OK
Applying auth.0002_alter_permission_name_max_length... OK
Applying auth.0003_alter_user_email_max_length... OK
Applying auth.0004_alter_user_username_opts... OK
Applying auth.0005_alter_user_last_login_null... OK
Applying auth.0006_require_contenttypes_0002... OK
Applying auth.0007_alter_validators_add_error_messages... OK
Applying auth.0008_alter_user_username_max_length... OK
Applying sessions.0001_initial... OK
```

```
root@38c80f8e29c3:/code# python manage.py makemigrations first_app
Migrations for 'first_app':
first_app/migrations/0001_initial.py
- Create model AccessRecord
- Create model Topic
- Create model Webpage
- Add field name to accessrecord
```

```
root@38c80f8e29c3:/code# python manage.py migrate
Operations to perform:
Apply all migrations: admin, auth, contenttypes, first_app, sessions
Running migrations:
Applying first_app.0001_initial... OK
```

### 4. 在对应app的admin.py中注册admin将要管理的模块：

```
from django.contrib import admin
from first_app.models import AccessRecord, Topic, Webpage
# Register your models here.
admin.site.register(AccessRecord)
admin.site.register(Topic)
admin.site.register(Webpage)
```

* 创建超级用户：（“root@38c80f8e29373147c3:/code” 说明当前主机是容器）

```
root@38c80f8e29c3:/code# python manage.py createsuperuser
Username (leave blank to use 'root'): ck
Email address: willxxx@gmail.com
Password:
Password (again):
Superuser created successfully.
```

### 5. 填充伪数据

```py
import os
# Configure settings for project
# Need to run this before calling models from application!
os.environ.setdefault('DJANGO_SETTINGS_MODULE','first_project.settings')

import django
# Import settings
django.setup()

import random
from first_app.models import Topic,Webpage,AccessRecord
from faker import Faker

fakegen = Faker()
topics = ['Search','Social','Marketplace','News','Games']

def add_topic():
t = Topic.objects.get_or_create(top_name=random.choice(topics))[0]
t.save()
return t



def populate(N=5):
'''
Create N Entries of Dates Accessed
'''

for entry in range(N):

# Get Topic for Entry
top = add_topic()

# Create Fake Data for entry
fake_url = fakegen.url()
fake_date = fakegen.date()
fake_name = fakegen.company()

# Create new Webpage Entry
webpg = Webpage.objects.get_or_create(topic=top,url=fake_url,name=fake_name)[0]

# Create Fake Access Record for that page
# Could add more of these if you wanted...
accRec = AccessRecord.objects.get_or_create(name=webpg,date=fake_date)[0]


if __name__ == '__main__':
print("Populating the databases...Please Wait")
populate(20)
print('Populating Complete')
```

* 执行populate\_first\_app.py

```
root@c271100f853b:/code# python populate_first_app.py
Populating the databases...Please Wait
Populating Complete
```

### 6. 编辑Views
+ 在 views 中 import 所有用到的 models
+ 用 view 查询 model 获取所需数据
+ 传送 model 结果到 template

```py
from django.shortcuts import render
# from django.http import HttpResponse
from first_app.models import Topic, Webpage, AccessRecord

# Create your views here.


def index(request):
my_dict = {'insert_me': "Hello I am from first_app/index.html!"}
return render(request, 'first_app/index.html', context=my_dict)


def help(request):
my_dict = {'insert_me': "Hello I am from first_app/help.html!"}
return render(request, 'first_app/help.html', context=my_dict)


def access(request):
webpages_list = AccessRecord.objects.order_by('date')
date_dict = {'access_records': webpages_list}
return render(request, 'first_app/access.html', context=date_dict)

```

### 7. 编辑 template 来接收来自 model 的数据并按设计的式样展示。

用模版标记语言 template tagging 链接模型和Html page

_Edit templates/first_app/access.html_


```html

<!DOCTYPE html>
{% load staticfiles %}
<html>
<head>
<meta charset="utf-8">
<title>Django Table</title>
<link rel="stylesheet" type="text/css" href="{% static "css/mystyle.css" %}">
<link rel="shortcut icon" href="{% static 'favicon/favicon.png' %}">
</head>
<body>
<h1>Hi welcome to Django MTV !</h1>
<h2>test<h2>
<div class="djangotwo">
{% if access_records %}
<table>
<thead>
<th>Site Name</th>
<th>Date Accessed</th>
</thead>
{% for acc in access_records %}
<tr>
<td>{{ acc.name }}</td>
<td>{{ acc.date }}</td>
</tr>
{% endfor %}
</table>
{% else %}
<p>NO ACCESS RECORDS FOUND!</p>
{% endif %}
</div>
</body>
</html>
```

### 8. 将相应的 URL 映射到新建的 View

_Edit first_app/urls.py_

```py
from django.conf.urls import url
from first_app import views

urlpatterns = [
url(r'^$', views.index, name='index'),
url(r'^help/', views.help, name='help'),
url(r'^access/', views.access, name='access'),
]

```

### 9. 模板的重用

+ 找出项目中重复出现的页面
+ 创建 base.html 添加 base tag
+ 在需要的地方扩展 base tag

### 在 admin 模版中判断用户分组

```html
    {% for group in request.user.groups.all %}
        {% ifequal group.name '报表' %}
            <h1 id="site-name"><a href="{% url 'production:index' %}">数据录入</a>
            </h1>
        {% else %}
            <h1 id="site-name">数据录入</h1>
        {% endifequal %}
    {% endfor %}
```

