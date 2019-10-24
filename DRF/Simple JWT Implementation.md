# Simple JWT Implementation

> [Github](https://github.com/davesque/django-rest-framework-simplejwt)



1. Install Pkg

```shell
pipenv install djangorestframework_simplejwt

# or add 'djangorestframework_simplejwt' to requirements.txt then:
pip install -r requirements.txt
```



2. Config Project settings to use Simple JWT

   ```python
   REST_FRAMEWORK = {
       ...
       'DEFAULT_AUTHENTICATION_CLASSES': (
           ...
           'rest_framework_simplejwt.authentication.JWTAuthentication',
       )
       ...
   }
   ```

   

3. Config `urls.py`

   ```python
   from rest_framework_simplejwt.views import (
       TokenObtainPairView,
       TokenRefreshView,
   )
   
   urlpatterns = [
       ...
       url(r'^api/token/$', TokenObtainPairView.as_view(), name='token_obtain_pair'),
       url(r'^api/token/refresh/$', TokenRefreshView.as_view(), name='token_refresh'),
       ...
   ]
   ```

   

4. Config Simple JWT settings

   ```python
   # Django project settings.py
   
   from datetime import timedelta
   
   ...
   
   SIMPLE_JWT = {
       'ACCESS_TOKEN_LIFETIME': timedelta(days=5),
       'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
       'ROTATE_REFRESH_TOKENS': False,
       'BLACKLIST_AFTER_ROTATION': True,
   
       'ALGORITHM': 'HS256',
       'SIGNING_KEY': SECRET_KEY,
       'VERIFYING_KEY': None,
   
       'AUTH_HEADER_TYPES': ('Bearer',),
       'USER_ID_FIELD': 'id',
       'USER_ID_CLAIM': 'user_id',
   
       'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
       'TOKEN_TYPE_CLAIM': 'token_type',
   
       'SLIDING_TOKEN_REFRESH_EXP_CLAIM': 'refresh_exp',
       'SLIDING_TOKEN_LIFETIME': timedelta(minutes=5),
       'SLIDING_TOKEN_REFRESH_LIFETIME': timedelta(days=1),
   }
   ```

   

5. Done