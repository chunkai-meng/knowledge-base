## 关于时区问题

> 今天是29/07/2023 NZTC

```py
from django.utils import timezone

from datetime import datetime

print(datetime.today().strftime("%d/%m/%Y"))
print(timezone.now().date().strftime("%d/%m/%Y"))
print(timezone.now().today().strftime("%d/%m/%Y"))
29/07/2023
28/07/2023
29/07/2023


In [10]: datetime.now()
Out[10]: datetime.datetime(2023, 7, 29, 10, 36, 19, 364151)

In [11]: datetime.now().date()
Out[11]: datetime.date(2023, 7, 29)

In [12]: timezone.localtime()
Out[12]: datetime.datetime(2023, 7, 29, 10, 41, 33, 414762, tzinfo=<DstTzInfo 'Pacific/Auckland' NZST+12:00:00 STD>)

In [13]: timezone.localtime().date()
Out[13]: datetime.date(2023, 7, 29)

```

datetime.now() 返回运行环境（我到MacOS）所设定到时区的时间。
date 是截取日期部分，所以是29号

timezone.now() 返回UTC 时间（通用时间），所以返回28号
很多时候model default 参数直接用 timezone.now 没关系是因为，数据库里存储到都是UTC时间，所以Django 页面读取了以后会根据settings USE_TZ=True 以及 TIMEZONE 来修正成当地时间显示出来

如果要在代码里获取当前时间(比如用于filter(date=current_date))就应该用： `timezone.localtime()`
获取当前日期：`timezone.localtime().date()`
```py
def date(self):  
    "Return the date part."  
    return date(self._year, self._month, self._day)
```
> 这里如果用 `datetime.now()` 以及 `datetime.now().date()`
在本地开发环境是可以的相等的。但不推荐，因为部署到生产的容器里，不确定有没有设定容器linux的timezone，就会导致`datetime.now().date()` 的结果是昨天或者明天了。


所以，重点不在于有没有时区。而是在于一个Python运行系统时区
一个给予Django settings 的时区，那当然是主动设置的Django settings 为准了。所以使用 timezone 最安全，但要记得 timezone.now() 是timezone = UTC，只有timezone.localtime()才是 timezone=settings.TIMEZONE

如何获取用户的当前日期来进行数据筛选？
> e.g. 服务器时间是29日，客户来自美国他的今天是28日，他查看今天的记录应该是看date==28的

```py
from django.utils import timezone
from datetime import datetime
import pytz
now = timezone.now()
# user_timezone_str = request.META.get('HTTP_TIMEZONE', 'UTC')
user_timezone_str = 'UTC'
user_timezone = pytz.timezone(user_timezone_str)
user_current_date = timezone.localtime(now, user_timezone).date()

print("user's today:", user_current_date)
```
> user's today: 2023-07-28

### 如果用today()
```py
print(datetime.today(), "-----", timezone.datetime.today())
# 2023-08-29 09:21:01.374636 ----- 2023-08-29 09:21:01.374643
```
就能得到相同的结果，它返回的date 对象其实还是包含时间的

2023-08-29 09:21:01.374636 ----- 2023-08-29 09:21:01.374643
```py
@classmethod  
def today(cls):  
    "Construct a date from time.time()."  
    t = _time.time()  
    return cls.fromtimestamp(t)
```


## 数据库

对于PostgreSQL，它默认存储UTC时间
> default, Django stores datetime values in the database in UTC and converts them to the local time zone when retrieved.

当我查询29日今天的记录的时候，它会给我新西兰时间是29日的记录，也就是created_at=（一部分28日的，一部分29日的记录）

如果此时一个美国用户打开我们的网站，查看他的当前（此刻他28日）记录
而你就此把,29日换成28日，其实也不会得到美国，当前的记录，
而是得到新西兰时间28日（昨天）的记录，依然是部分重合而已。
因为服务器Django并不关心
```py
Record.objects.filter(created_at__date="28/07/2023")
```
这里的28 是哪个时区，都会默认为settings.TIMEZONE 的时区。
所以真的要得到用户他当地当日发生的记录，只能用
```
Record.objects.filter(created_at__gt=<美国28日0时0分换算成新西兰时间>, created_at__lt=<美国29日0是0分换算成新西兰的时间>)
```
这样就能得到世界各国想看到的他的当天数据。


保存的时候也是这样的，如果用CBV，ModelForm，不指定template（默认），我猜测Django自动生成的form里面的日期字段在你post回服务器的时候会自动包含时区信息。
但如果自己写了def post(), 读取了 POST.get("date").
自己save/create model的话，Django会当作是新西兰时间。转换为UTC后保存到数据库。
