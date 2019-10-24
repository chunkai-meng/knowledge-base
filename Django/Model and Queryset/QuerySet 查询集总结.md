# QuerySet 查询集总结

[TOC]

## formset [[refer]](https://docs.djangoproject.com/en/2.1/topics/forms/modelforms/#model-formsets)

## 基于日期检索，快速创建 List View 
### 创建View
### 列出所有日期
### 在template中放置链接

## 筛选条件
`filter()` 函数接受多种 *关键字参数* (用逗号隔开) 作为筛选依据
参数名常用举例：

```python
fieldname__lte=数    # ≤
fieldname__gte=数    # ≥
fieldname__lt=数     # <
fieldname__gt=数     # >
fieldname(DateTime)__year=2019
fieldname(ForeignKey)__Field=='Good'
fieldname(ForeignKey)__Field__lte=0
fieldname__isnull=True
fieldname__in=[1, 3, 5, 7]
fieldname__startswith='why'
```

```python
# 复合条件判断
Post.objects.filter(publish__year__gte=2018, author__username='admin')

# 区间条件判断
Sample.objects.filter(date__range=["2011-01-01", "2011-01-31"])
```

‘exclude()’

## 增加查询结果

### 增加**列**（字段）
### 增加“合计”
```python
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