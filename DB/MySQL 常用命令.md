# MySQL常用命令
## 表：
+ 创建表：
```
create table 表名(列1 类型, 列2 类型...)
```

+ 使用旧表创建新表：

```
create table 表名 like 旧表名
create table 表名 as select 列1, 列2... from 旧表 definition only
删除表：
drop table 表名
```


- 显示所有表：

```
show tables
```


+ 显示表详情：

```
desc 表名
show create table 表名
```

+ 清空表：
```
truncate table 表名
delete from 表名
```

+ 修改表名：
```
alter table 原名 rename to 新名
```

添加列：
alter table 表名 add column 列名 类型

删除列：
alter table 表名 drop column 列名

修改列名：
alter table 表名 change 原名 新名 类型

修改列类型：
alter table 表名 modify 列名 类型

更新数据：
update 表名 set 字段=值 where 条件

删除数据：
delete from 表名 where 条件

插入数据：
insert into 表名(字段1, 字段2...) values(值1, 值2...)

排序：
select * from 表名 order by 字段 [desc|asc]

计数行：
select count(*) from 表名

## 数据库：
+ 创建数据库：
```
create database 库名
```

+ 删除数据库：
```
drop database 库名
```

+ mysql命令导入\导出表结构或数据:
```
# 1.导出整个数据库
mysqldump -u用户名 -p密码  数据库名 > 导出的文件名 
mysqldump -h 127.0.0.1 -uroot -ppassword database products > products.sql

# 2.导入数据库 
常用source 命令 
进入mysql数据库控制台， 
如mysql -u root -p 
mysql>use 数据库 
然后使用source命令，后面参数为脚本文件(如这里用到的.sql) 
source ./products.sql

# 3.导出一个表，包括表结构和数据 
mysqldump -u用户名 -p 密码  数据库名 表名> 导出的文件名 
C:\Users\jack> mysqldump -uroot -pmysql sva_rec date_rec_drv> e:\date_rec_drv.sql 

# 4.导出一个数据库结构 
C:\Users\jack> mysqldump -uroot -pmysql -d sva_rec > e:\sva_rec.sql 

# 5.导出一个表，只有表结构 
mysqldump -u用户名 -p 密码 -d数据库名  表名> 导出的文件名 
C:\Users\jack> mysqldump -uroot -pmysql -d sva_rec date_rec_drv> e:\date_rec_drv.sql 


```

- 重置 root 密码
```
# 不知道原始密码的情况下：
mysql.server stop
mysqld --skip-grant-tables &
```

```
# 新开一个终端
# 任意密码登陆
mysql -u root -p
UPDATE mysql.user SET authentication_string=PASSWORD('123') where User='root';
flush privileges;
```
Ctrl+C 退出之前的mysqld

```
mysql.server start
mysql -u root -p123
```

```
# 知道原始密码
mysql> use mysql;
Database changed
mysql> update user set password=password('yourpass') where user='root';
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```


# MySQL数据类型

- 数字类型
  - 整数: tinyint、smallint、mediumint、int、bigint
  - 浮点数: float、double、real、decimal
- 日期和时间: date、time、datetime、timestamp、year
- 字符串类型
  - 字符串: char、varchar
  - 文本: tinytext、text、mediumtext、longtext
- 二进制(可用来存储图片、音乐等): tinyblob、blob、mediumblob、longblob

## 字符串类型

| 类型 | 单位 | 最大 | 特性 |
| ---- | ----  | ---- | ---- |
| CHAR | 字符 | 最大为255字符 | 存储定长，容易造成空间的浪费 |
| VARCHAR | 字符 | 可以超过255个字符 | 存储变长，节省存储空间 |
| TEXT | 字节 | 总大小为65535字节，约为64KB | - |

- TEXT在MySQL内部大多存储格式为溢出页，效率不如CHAR
- Mysql默认为utf-8，那么在英文模式下1个字符=1个字节，在中文模式下1个字符=3个字节。

## 数字类型

### 整形

| type | Storage | Minumun Value | Maximum Value|
| :------------- | :------------- | :------------- | :------------- |
||(Bytes)|(Signed/Unsigned)|(Signed/Unsigned)|
|TINYINT|1|-128|127|
|||0|255|
|SMALLINT|2|-32768|32767|
|||0|65535|
|MEDIUMINT|3|-8388608|8388607|
|||0|16777215|
|INT|4|-2147483648|2147483647|
|||0|4294967295|
|BIGINT|8|-9223372036854775808|9223372036854775807|
|||0|18446744073709551615|


### 浮点型

| 属性 | 存储空间 | 精度 | 精确性 | 说明 |
| ---- | ----  | ---- | ---- | ---- |
|FLOAT(M, D)|4 bytes|单精度|非精确| 单精度浮点型，m总个数，d小数位 |
|DOUBLE(M, D)|8 bytes|双精度|比Float精度高| 双精度浮点型，m总个数，d小数位 |

- FLOAT容易造成精度丢失

### 定点数DECIMAL

- 高精度的数据类型，常用来存储交易相关的数据
- DECIMAL(M,N).M代表总精度，N代表小数点右侧的位数（标度）
- 1 < M < 254, 0 < N < 60;
- 存储空间变长


## 时间类型

| 类型 | 字节 | 例 | 精确性 |
| ---- | ----  | ---- | ---- |
| DATE | 三字节 | 2015-05-01 | 精确到年月日 |
| TIME | 三字节 | 11:12:00 | 精确到时分秒 |
| DATETIME | 八字节 | 2015-05-01 11::12:00 | 精确到年月日时分秒 |
| TIMESTAMP |  | 2015-05-01 11::12:00 | 精确到年月日时分秒 |

- MySQL在`5.6.4`版本之后，`TIMESTAMP`和`DATETIME`支持到微妙。
- `TIMESTAMP`会根据系统时区进行转换，`DATETIME`则不会
- 存储范围的区别  
    - `TIMESTAMP`存储范围：1970-01-01 00::00:01 to 2038-01-19 03:14:07
    - `DATETIME`的存储范围：1000-01-01 00:00:00 to 9999-12-31 23:59:59
- 一般使用`TIMESTAMP`国际化
- 如存时间戳使用数字类型`BIGINT`