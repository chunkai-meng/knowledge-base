# Getting Started with PostgreSQL on Mac OSX

```shell
$ brew install postgresql
```


```shell
pg_ctl -D /usr/local/var/postgres start && brew services start postgresql
```

Start Configuring Postgres

```shell
psql postgres
```

Show Users

```psql
postgres=# \du
```

Generate a [random password](Generate Random Passwords from the Command Line)

```shell
date |md5 | head -c12; echo
```

Create User postgres

```
CREATE ROLE postgres WITH LOGIN PASSWORD '6a9f09de726b'
```

```sql
postgres=# CREATE DATABASE infoma_dev;
postgres=# GRANT ALL PRIVILEGES ON DATABASE infoma_dev TO postgres;
postgres=# \q
```

Import  to *infoma_dev* DB

```shell
psql -h 127.0.0.1 -U postgres -W -p 5432 -d postgres -f 201910090001_postgres.sql
```

Enter PostreSQL and change DB

```shell
psql postgres
=# \c
=# \c infoma_dev
```

Install PostGIS [Extension](https://postgis.net/install/)

```
CREATE EXTENSION postgis;
ALTER EXTENSION postgis UPDATE;
```



#Knowledge/PostgreSQL

