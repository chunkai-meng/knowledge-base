# Deploy Django Project

![image-20190511112347722](https://img.shields.io/static/v1.svg?logo=Python&label=Python&message=3.7&color=informational)  ![image-20190511112347722](https://img.shields.io/static/v1.svg?logo=Django&label=Django&message=2.1.7&color=brightgreen) ![made-with-python](https://img.shields.io/badge/Make%20with-Docker%20Compose-077CBC.svg)![made-with-python](https://img.shields.io/static/v1.svg?logo=Nginx&label=Docker&message=Official%20Image&color=#099CEC) ![made-with-python](https://img.shields.io/static/v1.svg?logo=PostgreSQL&label=Docker&message=Official%20Image&color=#099CEC)




```shell
# remove any stopped containers and all unused images (not just dangling images)
docker system prune -a
```

## 1. Create Django container
**这里有3个重要文件**

> 1. requirements.txt
> 2. Dockerfile
> 3. uwsgi.ini


1. 生成requirements.txt

transform Pipfile to requirements (if use pipenv)
```sh
pipenv install pipenv-to-requirements
# To generate frozen requirements (ie, all dependencies have their version frozen)
pipenv run pipenv_to_requirements -f
```

2. Dockerfile
暴露出8888端口是 uwsgi 提供给 nginx 用的
```dockerfile
# ./api/Dockerfile
FROM python:3.7
LABEL maintainer CK 
ENV PYTHONUNBUFFERED 1
RUN mkdir /docker_api
WORKDIR /docker_api
COPY ./requirements.txt /docker_api
RUN pip install -r requirements.txt
EXPOSE 8888
```

3. uwsgi 配置文件
```python
# ./api/uwsgi.ini
[uwsgi]

http = :8888
chdir = /docker_api
wsgi-file = untitled/wsgi.py
processes = 2
threads = 1
```

4. docker-compose.yml
```yml
version: '3'
services:
        
    api:
        container_name: apicontainer
        build: ./api        
        restart: always
        # command: uwsgi  --emperor uwsgi.ini         
        command: uwsgi /docker_api/uwsgi.ini
        ports:
        - "8002:8000"
        - "8003:8888"
        volumes:
        - ./api:/docker_api
```

### Use MSSQL
```
pipenv install django-pyodbc-azure
```

```dockerfile
# Dockerfile
FROM python:3.7
LABEL maintainer CK 
ENV PYTHONUNBUFFERED 1
RUN mkdir -p /docker_api/www
WORKDIR /docker_api

COPY ./Pipfile ./Pipfile.lock /docker_api/
COPY ./static/images/favicon.ico /docker_api/www/

# django-pyodbc-azure need unixodbc-dev
RUN apt-get update && apt-get install -y unixodbc-dev
RUN pip install --upgrade pip
RUN python3 -m pip install pipenv
RUN pipenv install --deploy --system --ignore-pipfile
```



## 2. Create Nginx Container
1. 在项目中新建 nginx 目录
2. nginx 配置文件
3. SSL证书文件
4. docker-compose.yml


1. `mkdir nginx`
2. `cd nginx; mkdir conf.d`
```nginx
# ./nginx/conf.d/my_nginx.conf

# the upstream component nginx needs to connect to
upstream uwsgi.com {
    server api:8888;
}

# configuration of the server
server {
    # the port your site will be served on
    listen    80;
    # index  index.html;
    # the domain name it will serve for
    # substitute your machine's IP address or FQDN
    server_name  infoplus.co.nz www.infoplus.co.nz test.infoplus.co.nz;

    # Configuraton for https[2]
    # http request auto redirect to https
    # return 301 https://$server_name$request_uri;

    charset     utf-8;

    client_max_body_size 75M;   # adjust to taste

    # Django media
    # location /media  {
    #     alias /docker_api/static/media;  # your Django project's media files - amend as required
    # }

    location /static {
        alias /docker_api/static; # your Django project's static files - amend as required
    }

    location /.well-known {
        alias /docker_api/cert/.well-known;
    }
    
    #location / {
    #    uwsgi_pass  uwsgi;        
    #    include     /etc/nginx/uwsgi_params; # the uwsgi_params file you installed         
    #}

    location / {
        proxy_pass http://uwsgi.com/;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    # Configuration for https[1]
    listen 443 ssl;
    ssl_certificate /etc/nginx/conf.d/fullchain.pem;
    ssl_certificate_key /etc/nginx/conf.d/privkey.pem;
    ssl_dhparam /etc/nginx/conf.d/dhparams.pem;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
}

```

3. SSL 证书
这样 my_nginx.conf 中指向的证书文件就有了
```shell
sudo cp /etc/letsencrypt/live/infoplus.co.nz/fullchain.pem /etc/letsencrypt/live/infoplus.co.nz/privkey.pem /etc/ssl/certs/dhparams.pem ./nginx/conf.d
```

4. docker-compose.yml

`/etc/nginx/conf.d` 是默认存放配置的地方，nginx 会自己去那里找
```yml
# ./nginx/docker-compose.yml
version: '3'
services:
    nginx:
        container_name: nginx-container
        image: nginx
        ports:
        - "80:80" 
        - "443:443"
        volumes:
        - ./log:/var/log/nginx
        - ./cert:/docker_api/cert
        - ./conf.d:/etc/nginx/conf.d
        networks:
        - api-container

networks:
    api-container:
        external:
            name: https_default
```

## 3. Config https

> 这里其实是需要先有了http 服务，并且certbot 可以将临时验证文件存放到 根目录/.well-known/acme-challenge
> 所以配置nginx的时候将 ./cert 映射到了 容器中，并且将URL /.well-known/...
> 指向 /cert/.well-known/...
> 这样样certbot才能验证我们对域名的真正所有权，进而为我们申请证书。
> 然后我们将证书放到容器中，并配置nginx使用他们。
```shell
# 1. Install Git
# apt update
# apt install git


# 2. 下载 certbot

git clone https://github.com/certbot/certbot

# cd certbot
cd ~/certbot
./certbot-auto --help

# if OSError: Command /opt/eff.org/certbot/venv/bin/python2.7 - setuptools pkg_resources pip wheel failed with error code 1c
# then run below two line
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"

# 3. 生成免费证书
# ./certbot-auto certonly --webroot --agree-tos -v -t --email 邮箱地址 -w 网站根目录 -d 网站域名
./certbot-auto certonly --webroot --agree-tos -v -t --email ck@infoplus.co.nz -w ~/https/nginx/cert -d infoplus.co.nz -d www.infoplus.co.nz -d test.infoplus.co.nz

# ./certbot-auto certonly --webroot --agree-tos -v -t --email ck@infoplus.co.nz -w ~/InfoPlus/nginx/cert/ -d www.infoma.co.nz -d infoma.co.nz -d test.infoma.co.nz

./certbot-auto certonly --standalone --email ck@theia.co.nz -d www.infoma.co.nz -d infoma.co.nz -d test.infoma.co.nz

docker run -it --rm --name certbot \
            -v "/data/ssl:/etc/letsencrypt" \
            -v "/home/ubuntu/InfoPlus/nginx/cert/:/var/www/html" \
            certbot/certbot certonly -n --no-eff-email --email ck@theia.co.nz --agree-tos --webroot -w /var/www/html -d infoma.co.nz
用你自己的信息替换其中的值。




# 4. 生成 dhparams

# openssl dhparam -out /etc/ssl/certs/dhparams.pem 2048 发生文件冲突，而且2048太慢改为：
openssl dhparam -out /tmp/dhparams.pem 1024


# 5. 备份
sudo cp /etc/letsencrypt/live/infoplus.co.nz/fullchain.pem /etc/letsencrypt/live/infoplus.co.nz/privkey.pem /tmp/dhparams.pem ./nginx/conf.d
```

5. 配置 Nginx  
打开 nginx server 配置文件加入如下设置：
```nginx
    # Configuration for https[1]
    listen 443 ssl;
    ssl_certificate /etc/nginx/conf.d/fullchain.pem;
    ssl_certificate_key /etc/nginx/conf.d/privkey.pem;
    ssl_dhparam /etc/nginx/conf.d/dhparams.pem;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
```

## 4. Django Collect Static Files

```python
# project_name/settings.py
# 当运行 python manage.py collectstatic 的时候
# STATIC_ROOT 文件夹 是用来将所有STATICFILES_DIRS中所有文件夹中的文件，以及各app中static中的文件都复制过来
# 把这些文件放到一起是为了用apache等部署的时候更方便
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')
```


```sh
python manage.py collectstatic
```

```nginx
# nginx/conf.d/my_nginx.conf
location /media  {
    root /docker_api/media;
}
 
location /static {
    root /docker_api/collected_static;
}
```

## Mange Nginx
**Kill nginx container**
```sh
docker kill -s HUP nginx-container
```

**Restart Nginx**
```sh
nginx -s reload
```

## SSL 证书过期了

```sh
# 1. Install Git
apt update
apt install git


# 2. 下载 certbot
cd ~/InfoPlus # 项目目录
git clone https://github.com/certbot/certbot
cd certbot
./certbot-auto --help

# if OSError: Command /opt/eff.org/certbot/venv/bin/python2.7 - setuptools pkg_resources pip wheel failed with error code 1c
# then run below two line
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"

# 3. 生成免费证书
# ./certbot-auto certonly --webroot --agree-tos -v -t --email 邮箱地址 -w 网站根目录 -d 网站域名
./certbot-auto certonly --webroot --agree-tos -v -t --email ck@infoplus.co.nz -w ~/InfoPlus/nginx/cert/ -d test.infoplus.co.nz

# 4. 生成 dhparams
sudo rm -f /tmp/dhparams.pem
# openssl dhparam -out /etc/ssl/certs/dhparams.pem 2048 发生文件冲突，而且2048太慢改为：
openssl dhparam -out /tmp/dhparams.pem 1024

# 5. 备份
sudo cp /etc/letsencrypt/live/test.infoplus.co.nz/fullchain.pem /etc/letsencrypt/live/test.infoplus.co.nz/privkey.pem /tmp/dhparams.pem ~/InfoPlus/nginx/conf.d
```
