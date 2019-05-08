# [TOC]



## 为 QuerySet 增加字段
### 增加“合计”
```py
# count same field values in a query
from django.db.models import Count

fieldname = 'myCharField'
MyModel.objects.values(fieldname)
    .order_by(fieldname)
    .annotate(the_count=Count(fieldname))
```

### 增加一个多种判断的结构
[Conditional Expressions](https://docs.djangoproject.com/en/2.1/ref/models/conditional-expressions/)

```python
return self.queryset.annotate(
            if_I_liked=Case(
                When(users_like__in=user_id, then=Value(True)),
                default=Value(False),
                output_field=BooleanField(),
            )).annotate(
            if_my_favorite=Case(
                When(my_favorite__in=user_id, then=Value(True)),
                default=Value(False),
                output_field=BooleanField(),
            )).order_by('id')
```


```python
# 有条件的统计 manytomany 的个数
my_favorites = self.get_queryset().annotate(
    num_comment=Count('comment', filter=Q(comment__enabled=True))
).filter(my_favorite__in=user_id)
```

#Develop/Django/QuerySet 