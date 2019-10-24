# How to customize activate_url on django-allauth

> 自定义Django账户激活邮件的模版和domain address

> [refers to](https://stackoverflow.com/questions/27984901/how-to-customize-activate-url-on-django-allauth/34249336#comment94285011_34249336)

```py
# settings.py

# 告诉allauth 自定义的 adapter 在哪里
ACCOUNT_ADAPTER = 'untitled.adapter.DefaultAccountAdapterCustom'

# 现这句在生产机和开发机上有所不同
URL_FRONT = 'http://localhost:8000/'

# 可以分离settings.py 后在production.py 上改写成
URL_FRONT = 'https://www.infoma.co.nz/'
```

```py
# project folder/adapter.py
# 这里跟上面的对应，能找到就是对的

from allauth.account.adapter import DefaultAccountAdapter
from django.conf import settings


class DefaultAccountAdapterCustom(DefaultAccountAdapter):

    def send_mail(self, template_prefix, email, context):
        context['activate_url'] = settings.URL_FRONT + \
            'account/confirm-email.html/' + context['key'] + '/'
        msg = self.render_mail(template_prefix, email, context)
        msg.send()
```
