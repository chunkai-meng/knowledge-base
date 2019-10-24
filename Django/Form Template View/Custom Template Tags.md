# Custom Template Tags
> 将金额改成万为单位

#Django/Tags


```
# 借用官网的目录结构说明一下：tag函数所在的文件位置
polls/
    __init__.py
    models.py
    templatetags/
        __init__.py
        poll_extras.py
    views.py
```

```python
# polls/poll_extras.py
from django import template

register = template.Library()


@register.filter()
def has_group(user, group_name):
    return user.groups.filter(name=group_name).exists()


@register.filter(is_safe=True)      # 可以自定义小数点
def ten_thousand(value, decimal_place):
    if Decimal(value) < 10000:
        return '<1'
    return round(value/10000, decimal_place)

```

```html
# your template.html

{% load poll_extras %}
<td>销售总额</td>
…
<td>¥ {{ total_amount|default_if_none:0|intcomma:False }}（{{total_amount|ten_thousand:1}} 万）</td>
```

## 公共的tags 放到公共app
```
# 目录结构
proj/
    __init__.py
    common/
        __init__.py
		  apps.py
        templatetags/
            __init__.py
            common_tags.py
```

```python
# common/apps.py
from django.apps import AppConfig


class CommonConfig(AppConfig):
    name = ‘common’
```

```python
# settings.py
INSTALLED_APPS = [
	…
    ‘apps.common.apps.CommonConfig’,
	…
]
```