# Django CORS Problem

Problem Desc:
Django works as an API backend, frontend use VUE.
The request was blocked by CORS policy because the header doesn't involve 'Access-Control-Allow-Origin'.

![image-20190903223919631](media/image-20190903223919631.png)


1. install [django-cors-headers](https://pypi.org/project/django-cors-headers/)

```shell
pipenv install django-cors-headers
```

2. add to settings.py
```python
INSTALLED_APPS = (
    ...
    'corsheaders',
    ...
)
```

3. add middleware to settings.py
```python
MIDDLEWARE = [  # Or MIDDLEWARE_CLASSES on Django < 1.10
    ...
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    ...
]
```
4. add your request source to whitelist
```python
# replace with your own frontend server address
CORS_ORIGIN_WHITELIST = (
    '<server address>'
)
```

5. restart service, done!

