# PostgreSQL backup and import


## 安装
### 数据库系统采用 docker 安装，此处略过
### MAC安装cli
```
brew uninstall --force libpq
```

```sh
brew install libpq
echo 'export PATH="/usr/local/opt/libpq/bin:$PATH"' >> ~/.zshrc

```

### Ubuntu 安装cli
```
# remove the old one
sudo apt-get remove postgresql-client-10

# add the repository
sudo tee /etc/apt/sources.list.d/pgdg.list <<END
deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
END

# get the signing key and import it
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# fetch the metadata from the new repo
sudo apt-get update

# install 
sudo apt-get install postgresql-11
```
[其他版本的官方指引](https://www.postgresql.org/download/linux/ubuntu/)


## 创建用户
```
# Change the user to postgres :
su - postgres

# Create User for Postgres
createuser testuser

# Create Database
createdb testdb

# Acces the postgres Shell
psql ( enter the password for postgressql)

# Provide the privileges to the postgres user
alter user testuser with encrypted password 'qwerty';
grant all privileges on database testdb to testuser;
```

## 创建数据库
```psql
# psql
CREATE DATABASE targetdb;
```

```sh
# 命令行添加数据库
createdb -h 127.0.0.1 -p 8432 -U postgres <targetdb>
```

## 连接数据库
```
psql "dbname=postgres host=127.0.0.1 user=postgres password=password123 port=8432"

# OR

psql -h 127.0.0.1 -p 8432 -U postgres <targetdb>
```

[psql Guide](http://postgresguide.com/utilities/psql.html)

## 复制数据库
### 复制本地数据库
```
CREATE DATABASE targetdb 
WITH TEMPLATE sourcedb;
```
### 复制远程数据库
```
# 如果网速够快，且数据库很小，可以使用管道复制，否则请用导入导出的方法
pg_dump --dbname=postgresql://<username>:<password>@<remote_host>:<port>/<dbname> | psql -h 127.0.0.1 -p 8432 -U postgres test1
```


## 导出/倒入数据库
### 导出
#### 单行命令
```
pg_dump -h 127.0.0.1 -p 8432 -U postgres -W password123 -d postgres > /tmp/db.sql

```
[pg_dump Guide](https://www.postgresql.org/docs/11/app-pgdump.html)

#### 导出脚本
```sh
BDIR=~/backup/

# 需要备份的数据库，可设置多个: mysql db1 db2
PG_BACKUP_DBS="postgres"

# 数据库密码，Docker环境变量在crontab执行脚本时不起作用，所以在此重新设置
PG_USERNAME=postgres
PG_PASSWORD=password123
PG_PORT=8432

# 删除7天前的备份
OLD_FILES=`date -v -7d`*

cd $BDIR
rm -rf $OLD_FILES

DATE=`date +%Y%m%d%H%M`

for DB_NAME in $PG_BACKUP_DBS
do
    FILE_PATH=${BDIR}${DATE}_${DB_NAME}.sql.gz
    # pg_dump --dbname=postgresql://postgres:password123@127.0.0.1:8432/postgres | gzip > /tmp/psql.gz
    pg_dump --dbname=postgresql://$PG_USERNAME:$PG_PASSWORD@127.0.0.1:$PG_PORT/${DB_NAME} | gzip > $FILE_PATH

    /bin/sleep 3
done

```

### 导入
#### 显示当前所在数据库
```
\c
```

#### 新建数据库（如有必要）
```
CREATE DATABASE test;
```

#### 切换当前使用的数据库
```
\c test;
```

#### 重命名/备份数据库
```
ALTER DATABASE postgres RENAME TO postgres_bk;
```

#### 新建数据库
```
CREATE DATABASE postgres;
```

#### 导入数据库
```
psql -h 127.0.0.1 -U postgres -W -p 8432 -d postgres -f ~/backup/201812282307_postgres.sql

# 管道倒入 SQL 压缩包
gunzip -c filename.gz | psql dbname
```


## 创建备份目录
```
mkdir /tmp/backup
```

## 创建备份容器
```
docker run --rm --volumes-from infoma-db -v /tmp/backup:/backup ubuntu:18.04 tar cvf /backup/infoma-db-volume.tar /var/lib/postgresql/data
```
各参数的作用
1.创建一个新的容器  `run .... ubuntu:18.04` 
2.挂载数据卷容器    `--volumes-from shiyanloudb`
3.挂载宿主机本地目录作为数据卷  `-v /tmp/backup:/backup`
4.将数据卷容器的内容备份到宿主机本地目录挂载的数据卷中  `tar cvf /backup/infoma-db-volume.tar /var/lib/postgresql/data`
5.完成备份操作后容器销毁    ` --rm`