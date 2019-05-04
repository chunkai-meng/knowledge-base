# Customizing JWT response from django-rest-framework-simplejwt
add extra fields to rest-auth login response

- **For example**: to customize simpleJWT response by adding username and groups,

# [![enter image description here][1]][1]

1. Override the `validate` method in `TokenObtainPairSerializer`

```python
# project/views.py

from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
from rest_framework_simplejwt.views import TokenObtainPairView


class MyTokenObtainPairSerializer(TokenObtainPairSerializer):
    def validate(self, attrs):
        data = super().validate(attrs)
        refresh = self.get_token(self.user)
        data['refresh'] = str(refresh)
        data['access'] = str(refresh.access_token)

        # Add extra responses here
        data['username'] = self.user.username
        data['groups'] = self.user.groups.values_list('name', flat=True)
        return data


class MyTokenObtainPairView(TokenObtainPairView):
    serializer_class = MyTokenObtainPairSerializer
```

2. replace the login view with customized view

```Python
# project/urls.py

from .views import MyTokenObtainPairView

urlpatterns = [
    # path('token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('token/', MyTokenObtainPairView.as_view(), name='token_obtain_pair'),
]
```

🔚

> **References：** [SimpleJWT Readme](https://github.com/davesque/django-rest-framework-simplejwt) and the source code as below: 
[![enter image description here][2]][2]


[1]: https://i.stack.imgur.com/rqSD7.png
[2]: https://i.stack.imgur.com/hygRr.png