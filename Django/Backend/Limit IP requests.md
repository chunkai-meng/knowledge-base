# Limit IP requests

```Python
# untitled/middlewares.py

from django.utils.deprecation import MiddlewareMixin
from django.http import HttpResponse
import datetime
from django.conf import settings
from rest_framework import status


# 模存入用户ip的访问情况
HISTORY = {}
# 模拟存放被禁的ip名单数据表
BLACK_LIST = []
LIMIT = settings.IP_REQUEST_LIMIT or {'interval': 60, 'times': 500}


class Throttle(MiddlewareMixin):
    def process_request(self, request):
        error_detail = 'Your IP address exceeds limit'
        # 拿到当前访问的时间
        now = datetime.datetime.now()
        # 拿到访问者的ip
        ip = request.META.get("REMOTE_ADDR")
        # 判断此ip有没有被记录在黑名单如，如果有不让它继续访问
        if ip in BLACK_LIST:
            return Response(status=status.HTTP_400_BAD_REQUEST)
        # 判断一下当前ip有没有访问的 history，如果没有就给它添加一个记录
        if ip not in HISTORY:
            HISTORY[ip] = {'last': now, 'times': 0}
        # 做访问次数的逻辑判断，在1分钟内次数加1
        if (now - HISTORY[ip]['last']) < datetime.timedelta(seconds=LIMIT['interval']):
            HISTORY[ip]['times'] += 1
        else:
            # 超过一分钟就清空之前的记录，重新计算
            HISTORY[ip]['times'] = 0
            HISTORY[ip]['last'] = now
        # 判断如果它的在一分钟之内，请求了1000次，将它的ip加入到黑名单内
        if HISTORY[ip]['times'] > LIMIT['times']:
            BLACK_LIST.append(ip)
            return HttpResponse(error_detail,
                                status=status.HTTP_429_TOO_MANY_REQUESTS)
```

```python
# untitled/settings/defaults.py

MIDDLEWARE = [
    ....
    'untitled.middlewares.Throttle',
]

# Throttle MiddleWare
IP_REQUEST_LIMIT = {'interval': 60, 'times': 2}
```

**References**

[CSDN - 服务端防止大规模的恶意请求](https://blog.csdn.net/miaoqinian/article/details/80826000)