# How to search date or datetime and related fields in Django Admin

## For date or datetime fields
1. date or datetime field is not in string type
2. **admin.search_fields** only support fields or related fields of string type
3. but still we can search date by overriding `get_search_results`
4. [see django docs](https://docs.djangoproject.com/en/2.1/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields)

## For related fields
user `related_field__<related_object's field>`

```python
# app/admin.py

class MaterialAdmin(admin.ModelAdmin):
    ....
    # Don't need to enter 'date'
    # 'supplier__name' is related fields
    search_fields = ('supplier__name', 'service_name')

    def get_search_results(self, request, queryset, search_term):
        queryset, use_distinct = super().get_search_results(request, queryset, search_term)
        try:
            import datetime as dt
            import pytz
            date_obj = dt.datetime.strptime(search_term, '%Y年%m月%d日').date()
            # date_obj = dt.datetime.strptime(search_term, '%Y-%m-%d').date() # choose your own date format
            # date_time_obj = dt.datetime.strptime(search_term,  '%Y-%m-%d %H:%M:%S') # if you want to search datetime field
        except ValueError:
            pass
        else:
            queryset |= self.model.objects.filter(date=date_obj)
            # queryset |= self.model.objects.filter(date=date_time_obj)     # if search datetime
        return queryset, use_distinct
```

*Tested in Django 2.1.7*