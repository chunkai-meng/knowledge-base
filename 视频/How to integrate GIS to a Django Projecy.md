# How to integrate GIS to a Django Projecy

## 时间轴

### Prerequirement
1. A MacOS Python3+
2. Python Virtual Environment
2. A Django Projec

### What we need to do
1. Installing GeoDjango Dependencies
2. Setting up a Spatial Database With PostgreSQL and PostGIS
3. Setting up Your Project

### Step1
1. Installing GeoDjango Dependencies

```shell
$ brew install postgis
$ brew install gdal
$ brew install libgeoip
```

### Step2
Setting up a Spatial Database With PostgreSQL and PostGIS

Generate random password (Optional)

```shell
date |md5 | head -c12; echo
```

```shell
docker run --name=postgis -d -e POSTGRES_USER=user001 -e POSTGRES_PASS=123456789 -e POSTGRES_DBNAME=gis -p 5432:5432 kartoza/postgis:11.0-2.5
```

### Step3
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

### Step4

应用，建一个Event App，然后按地址排序

**Model**

```python
# apps/event/models.py

from django.contrib.gis.db import models
from django.contrib.gis.geos import Point

class Event(BaseModel):
    name = models.CharField(max_length=100)
    desciption = MartorField()    
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

