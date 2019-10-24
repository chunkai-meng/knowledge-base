# Verifying user's group
> Check if a user is in a specific group

**Step 1: register a  tag**
```python
# app/templatetags/has_group.py

from django import template

register = template.Library()

@register.filter(name='has_group')
def has_group(user, group_name):
    return user.groups.filter(name=group_name).exists()
```

**Step 2: implement the tag to your template**

```html
# proj/templates/admin/base_site.html

{% extends "admin/base_site.html" %}
{% load has_group %}

{% block branding %}
    {% if request.user|has_group:"报表" %}
        <h1 id="site-name">
            <a href="{% url 'production:transaction-list' %}">数据录入</a>
        </h1>
    {% else %}
        <h1 id="site-name">数据录入</h1>
    {% endif %}

{% endblock %}
```