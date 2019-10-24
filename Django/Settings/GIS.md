# Make a Location-Based Web App With Django and GeoDjango

[TOC]




## Create Local Dev environment
for debugging on my Mac


```shell
brew install postgresql
brew install postgis
brew install gdal
brew install libgeoip
```

## Create Docker Image
for the future server we need to test the new image on local Mac first

### New DB Image - PostgreSQL

Create a new local Images DB: PostgreSQL Image** - specific for PostGIS

```yaml
version: '3.6'
services:
  db-gis:
    image: kartoza/postgis
    container_name: infoma-db-gis
    restart: always
    environment:
      POSTGRES_USER: '${POSTGRES_USER}'
      POSTGRES_PASSWORD: '${POSTGRES_PASSWORD}'
    ports:
      - '8432:5432'
    volumes:
      - pgdata:/var/lib/postgresql/data/
    networks:
      - db-net

  api-gis:
    container_name: infoma-api-gis
    build:
      context: ./api
      dockerfile: ./Dockerfile.GIS
    restart: always
    environment:
      DJANGO_SETTINGS_MODULE: untitled.settings
    command: uwsgi /docker_api/uwsgi.ini
    volumes:
      - ./api:/docker_api
    networks:
      - backend
      - db-net
    depends_on:
      - db-gis
```

```shell
# Postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password123
PGADMIN_DEFAULT_EMAIL=ck@theia.co.nz
PGADMIN_DEFAULT_PASSWORD=password123
```

GIS PostgreSQL DB Image is from Docker Hub not need to Build


**Migrate Online DB to New PostGIS** - *see if infoma show correctly*

Backup from Server

```shell
pg_dump --dbname=postgresql://$PG_USERNAME:$PG_PASSWORD@127.0.0.1:$PG_PORT/${DB_NAME} | gzip > $FILE_PATH
```

Download

```shell
scp infoma:backup/201910090001_postgres.sql.gz /tmp
```

Import to local test DB

```shell
psql -h 127.0.0.1 -U postgres -W -p 8432 -d postgres -f /tmp/201910090001_postgres.sql
```

* [x] Check the data via GUI Tool

### New API Image - Django

Add install package `RUN` to Dockerfile

```Dockerfile
FROM python:3.7
LABEL maintainer CK 
ENV PYTHONUNBUFFERED 1
RUN mkdir -p /docker_api/www
WORKDIR /docker_api


# Using pip:
# COPY ./requirements.txt /docker_api
# RUN pip install -r requirements.txt

# Using pipenv:
COPY ./Pipfile ./Pipfile.lock /docker_api/
COPY ./static/imgs/favicon.ico /docker_api/www/
RUN python3 -m pip install pipenv
# --system install all packages into the system python, and not into the virtualenv
# --deploy flag, so your build will fail if your Pipfile.lock is out of date
# --ignore-pipfile, Ignore Pipfile when installing, using the Pipfile.lock.so it won't mess with our setup
RUN pipenv install --deploy --system --ignore-pipfile

# <----Down below---->
RUN sudo aptitude install gdal-bin libgdal-dev python3-gdal binutils libproj-dev

EXPOSE 8888
```

Build

```
docker-compose up --build api-gis
```

Add Tag

```shell
docker tag efc4f078437f hustmck/infoma_api-gis:1.0
```

Push to Docker Hub

```shell
docker push hustmck/infoma_api-gis:1.0
```

## Backup Database

## Backup DB Volumes

## User New Image in Server

```yaml
version: '3.6'
services:
  infoma-db:
    image: postgres
    container_name: infoma-db
    restart: always
    environment:
      POSTGRES_USER: '${POSTGRES_USER}'
      POSTGRES_PASSWORD: '${POSTGRES_PASSWORD}'
    ports:
      - '8432:5432'
    volumes:
      - pgdata:/var/lib/postgresql/data/
    networks:
      - db-net

  api:
    # build: ./api
    image: hustmck/infoma_api:enablecaching
    container_name: infoma-api
    restart: always
    environment:
      DJANGO_SETTINGS_MODULE: untitled.settings.production
    command: uwsgi /docker_api/uwsgi.ini
    volumes:
      - ./api:/docker_api
    networks:
      - backend
      - db-net
    depends_on:
      - infoma-db
      - redis
```

Start Service

```
docker-compose up loc-api
```

## References
- [**macOS preparation GeoDjango**](https://docs.djangoproject.com/en/2.1/ref/contrib/gis/install/#macos)

- [Make a Location-Based Web App With Django and GeoDjango](https://realpython.com/location-based-app-with-geodjango-tutorial/)