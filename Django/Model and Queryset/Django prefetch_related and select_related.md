# Django prefetch_related and select_related

> Created: Aug 02, 2019 9:40 PM
Last Edited Time: Aug 03, 2019 5:11 PM
Status: Draft

# Description

Boost ORM performance with prefetch_related() and select_related()

## Show SQL

### Django-extensation

1. Use django-extensation to show SQL and Execution time

        pipenv install django-extensions --dev

2. Add to INSTALL_APPS

        # proj.settings.dev.py
        INSTALLED_APPS = INSTALLED_APPS + ['django_extensions']

3. Enter Django Shell with SQL printed

        ./manage.py shell_plus --print-sql --settings proj.settings.dev

![Untitled-cdd84f77-1cc1-4a2f-987c-d1a9583d0ffe](media/15664226967875/Untitled-cdd84f77-1cc1-4a2f-987c-d1a9583d0ffe.png)

### Change LOGGING settings

good for server log watching, not recommanded for django interactive console.

    # proj.settings.dev.py
    
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'handlers': {
            'console': {
                'class': 'logging.StreamHandler',
            }
        },
        'loggers': {
            'django.db.backends': {
                'handlers': ['console'],
                'propagate': True,
                'level': 'DEBUG',
            }
        },
    }

# Finding

- Model.objects.all() will SELECT all it related object as well, lead to a multiple SELECT
- So we should use Model.objects.values('', '',...) to skip the foreignkey field

# select_related

用于一对一，或者多对一（foreignkey field）总之是对一就好了，找出来加到查询结果了面成为额外字段

- 用于一对一，或者多对一（foreignkey field）总之是对一就好了，找出来加到查询结果了面成为额外字段

# prefetch_related

用于多对一（foreignkey 的related 字段）或者多对多 ManyToMany，增加的属性是一个列表

-