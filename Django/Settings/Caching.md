# Caching in Django With Redis

**References**

[Caching in Django With Redis](https://realpython.com/caching-in-django-with-redis/)
[Django’s cache framework](https://docs.djangoproject.com/en/2.2/topics/cache/)

Start Caching Server

```shell
docker run --name redis --rm -p 127.0.0.1:6379:6379 -d redis
```


pipenv install django-redis


Edit settings

```python
# settings/caching.py

from .defaults import *
DEBUG = True

CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/2",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient"
        },
        "KEY_PREFIX": "infoma"
    }
}
```


```python
# apps/common/viewsets.py

from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page


class CommonInfoViewSet(viewsets.ModelViewSet):
    @method_decorator(cache_page(60))
    def list(self, request, *args):
```