 - На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
 - Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2. 
 - На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение. 
 - Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1. 
 - 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). 
 - Небольшое описание, того, что получилось.

Создал 3 машины, установил на все 3 постгрес 13, настроил подключения внутри облака и создал на все трёх одного суперпользователя:
```
CREATE USER repl WITH SUPERUSER  PASSWORD '***';
```
Ну и подключался:
```
psql -h 10.129.0.20 -U repl -d postgres
```

Первые 2 машины перевёл в ALTER SYSTEM SET wal_level = logical;<BR>
 Создали таблички и публикацию (на второй машине test2):
 ```
 postgres=# create table test(a int);
CREATE TABLE
postgres=# create table test2(a int);
CREATE TABLE
postgres=# CREATE PUBLICATION test_pub FOR TABLE test;
CREATE PUBLICATION
 
postgres=# CREATE PUBLICATION test_pub3 FOR TABLE test;
CREATE PUBLICATION
```
 
 Взаимно подписался:
 ```
postgres=# CREATE SUBSCRIPTION test_sub
postgres-# CONNECTION 'host=10.129.0.26 port=5432 user=repl password=*** dbname=postgres'
postgres-# PUBLICATION test_pub WITH (copy_data = true);
NOTICE:  created replication slot "test_sub" on publisher
CREATE SUBSCRIPTION
 ```
 ```
 postgres=# CREATE SUBSCRIPTION test_sub
postgres-# CONNECTION 'host=10.129.0.20 port=5432 user=repl password=*** dbname=postgres'
postgres-# PUBLICATION test_pub WITH (copy_data = true);
NOTICE:  created replication slot "test_sub" on publisher
CREATE SUBSCRIPTION
```
postgres=# CREATE SUBSCRIPTION test_sub1
postgres-# CONNECTION 'host=10.129.0.20 port=5432 user=repl password=jsfnxncdsnfsnDSNFj3dj dbname=postgres'
postgres-# PUBLICATION test_pub3 WITH (copy_data = true);
NOTICE:  created replication slot "test_sub1" on publisher
CREATE SUBSCRIPTION
postgres=# CREATE SUBSCRIPTION test_sub2
postgres-# CONNECTION 'host=10.129.0.26 port=5432 user=repl password=jsfnxncdsnfsnDSNFj3dj dbname=postgres'
postgres-# PUBLICATION test_pub3 WITH (copy_data = true);
NOTICE:  created replication slot "test_sub2" on publisher
CREATE SUBSCRIPTION
```
 
 Проверяем:<BR>
На первой машине:
```
 postgres=# insert into test values(1), (3);
INSERT 0 2
```
 
 На второй машине:
 ```
 postgres=# select * from test;
 a
---
 1
 3
(2 rows)
```
 На третьей машине:
 ```
 postgres=# select * from test;
 a
---
 1
 3
(2 rows)
```
 На второй машине
 ```
 postgres=# insert into test2 values(2), (4);
INSERT 0 2
```
 На первой машине
 ```
 postgres=# select * from test2;
 a
---
 2
 4
(2 rows)
```
 
 На третьей машине:
 ```
 postgres=# select * from test2;
 a
---
 2
 4
(2 rows)
 ```
