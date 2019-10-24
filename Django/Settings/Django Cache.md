# Django Cache

#incomplete

[Database caching](https://docs.djangoproject.com/en/2.0/topics/cache/)

## 设置缓存
+ setting 文件中的 CACHES 项可以设置所有变量值
+ 常用模块：python-memcached and pylibmc两个模块


### Memcached
+ 将 BACKEND 设置为django.core.cache.backends.memcached.MemcachedCache 或者 django.core.cache.backends.memcached.PyLibMCCache (取决于你所选绑定memcached的方式)
+ 将 LOCATION 设置为 ip:port 值，ip 是 Memcached 守护进程的ip地址， port 是Memcached 运行的端口。或者设置为 unix:path 值，path 是 Memcached Unix socket file的路径.
    ```
    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
            'LOCATION': '127.0.0.1:11211',
        }
    }
    ```

### Database caching

+ 把BACKEND设置为django.core.cache.backends.db.DatabaseCache
+ 把 LOCATION 设置为 tablename，数据表的名称。这个名字可以是任何你想要的名字，只要它是一个合法的表名并且在你的数据库中没有被使用过。
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}
```

## Filesystem caching
### Local-memory caching 本地内存缓存

## 站点级缓存