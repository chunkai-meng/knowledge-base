# How to integrate GIS to a Django Projecy

**时间轴**

- 介绍最终效果

一张图看到我们可以添加地理位置字段，并且在admin site 管理

还可以根据与给定位置之间的距离长短排序，这也是配置GeoDjango的主要用途，后面会详细介绍。

- Prerequirement

1. A MacOS Python3+
2. Python Virtual Environment
2. A Django Projec

- What we need to do

1. Installing GeoDjango Dependencies
2. Setting up a Spatial Database With PostgreSQL and PostGIS
3. Setting up Your Project

- Step1
1. Installing GeoDjango Dependencies

```shell
$ brew install postgis
$ brew install gdal
$ brew install libgeoip
```

- Step2
Setting up a Spatial Database With PostgreSQL and PostGIS

Generate random password (Optional)

```shell
date |md5 | head -c12; echo
```

```shell
docker run --name=postgis -d -e POSTGRES_USER=user001 -e POSTGRES_PASS=123456789 -e POSTGRES_DBNAME=gis -p 5432:5432 kartoza/postgis:11.0-2.5
```

- Step3
Change Django Settings


```python
DATABASES = {
    'default': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis',
        'NAME': 'gis',
        'USER': 'user001',
        'PASSWORD': '123456789',
        'HOST': 'localhost',
        'PORT': '5432'
    }
}
```

```python
INSTALLED_APPS = [
    # [...]
    'django.contrib.gis'
]
```

- Step4

应用，建一个Event App，然后按地址排序

```shell
./manage.py start app event
```

```python
INSTALLED_APPS = [
    # [...]
    'django.contrib.gis',
    'event',
]
```

**Model**

```python
# apps/event/models.py

from django.contrib.gis.db import models
from django.contrib.gis.geos import Point

class Event(BaseModel):
    name = models.CharField(max_length=100)
    desciption = models.TextField(blank=True)
    location = models.PointField()
    address = models.CharField(max_length=100)

    def __str__(self):
        return self.name
```



**Admin**

```python
# apps/event/admin.py

from django.contrib.gis.admin import OSMGeoAdmin
from .models import Event


@admin.register(Event)
class EventAdmin(OSMGeoAdmin):
    list_display = ('id', 'name', 'description', 'location', 'address')

```

```shell
$ python manage.py makemigrations
$ python manage.py migrate
$ python manage.py createsuperuser
$ python manage.py runserver
```

**View**
```python
from django.views import generic
from django.contrib.gis.geos import fromstr
from django.contrib.gis.db.models.functions import Distance
from .models import Event

longitude = -80.191788
latitude = 25.761681
user_location = Point(longitude, latitude, srid=4326)

class Home(generic.ListView):
    model = Event
    context_object_name = 'Event'
    queryset = Event.objects.annotate(distance=Distance('location',
    user_location)
    ).order_by('distance')[0:6]
    template_name = 'event/index.html'
```

**Template**

```html
{% extends "admin/base_site.html" %}

{% block content %}
    {% if shops %}
    <ul>
    {% for shop in shops %}
        <li>
        {{ shop.name }}: {{shop.distance}}
        </li>
    {% endfor %}
    </ul>
    {% endif %}
{% endblock %}
```

```python
from django.urls import path
from shops import views

urlpatterns = [
    # [...]
    path('', views.ShopList.as_view())
]
```
