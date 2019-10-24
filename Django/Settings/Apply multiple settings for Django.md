# Apply multiple settings for Django

> 将settings 分为dev 和prod

**1. 在 settings.py 所在目录新建 `settings` folder**

Create __init__.py in settings folder with below line
```py
# __init__.py

from .defaults import *
```

**2. 将 settings.py 移动到  `settings` folder 的 default.py**
```
mv settings.py settings/defaults.py
```

**3. In `settings`**
Create loc.py
```
from .defaults import *

DEBUG = True

ALLOWED_HOSTS = ['celery',
                 'test.infoplus.co.nz',
                 'test.infoma.co.nz',
                 'infoma.co.nz']
```


**4. Create prod.py**
```
from .defaults import *

DEBUG = False

ALLOWED_HOSTS = ['celery',
                 'infoma.co.nz',
                 'test.infoma.co.nz',
                 ]

```

**5. Add `BASE_DIR = os.path.join(BASE_DIR, '..')` to defaults.py**

After adding should be as below

```
# defaults.py

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# After move settings into a deeper folder
BASE_DIR = os.path.join(BASE_DIR, '..')
# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/
```

6. Tell docker-compose you changed django setting location

```yml
# docker-compose.yml

version: '3.6'
services:

  celery:
    ....
    environment:
      DJANGO_SETTINGS_MODULE: proj.settings.prod
    ....
```

**7. Tell uwsgi**

Since docker's already set up `DJANGO_SETTINGS_MODULE` env in container, I just comment out the corresponding line.

Let docker-compose.yml decide which setting to use.

```
# import os

from django.core.wsgi import get_wsgi_application

# os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.settings')

application = get_wsgi_application()

```
