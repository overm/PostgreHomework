 - создайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL)

Я создал кластер <B>GreenPlum</B>, так как у меня цель обучения познакомиться именно с ним. По сути отличия в ДЗ только в том, что пользователь gpadmin и писать в таблицах поле распределение distributed by. С разворачиванием кластера GP пришлось немного повозиться с ssh и bash'ем и конф. файлами.

 - зайдите в созданный кластер под пользователем postgres

gpadmin@gpm:/home/pavel $ psql postgres<BR>
psql (9.4.26)<BR>
Type "help" for help.<BR>
<BR>
postgres=#<BR>

 - создайте новую базу данных testdb
  
postgres=# create database testdb;<BR>
CREATE DATABASE

 - зайдите в созданную базу данных под пользователем postgres
  
postgres=# \c testdb<BR>
You are now connected to database "testdb" as user "gpadmin".

 - создайте новую схему testnm

testdb=# create schema testnm;<BR>
CREATE SCHEMA

 - создайте новую таблицу t1 с одной колонкой c1 типа integer

 Сделаем вид, что мы не заметили, что создаём в схеме public<BR>
testdb=# create table t1(c1 int) distributed by (c1);<BR>
CREATE TABLE<BR>

 - вставьте строку со значением c1=1
  
testdb=# insert into t1(c1) values(1);<BR>
INSERT 0 1<BR>
<BR>
testdb=# select * from <B>public</B>.t1;<BR>
 c1<BR>
----<BR>
  1<BR>
(1 row)<BR>

 - создайте новую роль readonly
 
 testdb=# create role readonly;<BR>
NOTICE:  resource queue required -- using default resource queue "pg_default"<BR>
CREATE ROLE<BR>
 
 - дайте новой роли право на подключение к базе данных testdb
 
testdb=# grant connect on database testdb to readonly;<BR>
GRANT

 - дайте новой роли право на использование схемы testnm
 
testdb=# grant usage on schema testnm to readonly;<BR>
GRANT

 - дайте новой роли право на select для всех таблиц схемы testnm
 
testdb=# grant select on all tables in schema testnm to readonly;<BR>
GRANT

 - создайте пользователя testread с паролем test123

testdb=# create role testread with login password '******';<BR>
NOTICE:  resource queue required -- using default resource queue "pg_default"<BR>
CREATE ROLE

 - дайте роль readonly пользователю testread
 
testdb=# grant readonly to testread;<BR>
GRANT ROLE

 - зайдите под пользователем testread в базу данных testdb

Чуть поправил и выполнил обновление hba_file<BR>
om@play-box:~$ psql -U testread -h 51.250.73.110 -d testdb<BR>
Password for user testread:<BR>
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 9.4.26)<BR>
Type "help" for help.<BR>
<BR>
testdb=><BR>
 
 
 - сделайте select * from t1;
 
testdb=> select * from t1;<BR>
ERROR:  permission denied for relation t1<BR>
 
 
 - получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
 
Таблица создалась в public, куда мы доступ не давали. 
 
 - напишите что именно произошло в тексте домашнего задания
 
 см выше
 
 - у вас есть идеи почему? ведь права то дали?
 
 см выше
 
 - посмотрите на список таблиц
 
testdb=> \dt<BR>
        List of relations<BR>
 Schema | Name | Type  |  Owner<BR>
--------+------+-------+---------<BR>
 <B>public</B> | t1   | table | gpadmin<BR>
(1 row)<BR>

 
 - подсказка в шпаргалке под пунктом 20
 - а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
 - вернитесь в базу данных testdb под пользователем postgres
 - удалите таблицу t1
 
 testdb=# drop table t1;<BR>
DROP TABLE

 
 - создайте ее заново но уже с явным указанием имени схемы testnm

 testdb=# create table testnm.t1(c1 int) distributed by (c1);<BR>
CREATE TABLE<BR>

 
 - вставьте строку со значением c1=1
 
testdb=# insert into testnm.t1(c1) values(1);<BR>
INSERT 0 1

 - зайдите под пользователем testread в базу данных testdb
 
om@play-box:~$ psql -U testread -h 51.250.73.110 -d testdb<BR>
Password for user testread:<BR>
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 9.4.26)<BR>
Type "help" for help.<BR>
 
 
 - сделайте select * from testnm.t1;
 
testdb=> select * from testnm.t1;<BR>
ERROR:  permission denied for relation t1

 
 - получилось?
 
 Нет
 
 - есть идеи почему? если нет - смотрите шпаргалку
 
 Права выданы на объекты, когда таблицы не существовало.
 
 - как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
 
testdb=# ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;
ALTER DEFAULT PRIVILEGES

 
 - сделайте select * from testnm.t1;
 
 testdb=> select * from testnm.t1;<BR>
ERROR:  permission denied for relation t1

 
 - получилось?
 
 нет
 
 - есть идеи почему? если нет - смотрите шпаргалку
 
 Потому что действует для новых таблиц<BR>
 Снова:<BR>
 testdb=# grant select on all tables in schema testnm to readonly;<BR>
GRANT

 
 - сделайте select * from testnm.t1;
 
testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)
 
 
 - получилось?
 
 Да
 
 - ура!
 - теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
 
testdb=> create table t2(c1 integer) distributed by(c1); insert into t2 values (2); <BR>
CREATE TABLE<BR>
INSERT 0 1<BR>

 
 - а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
 
 Снова создали в паблике, а права на паблик есть изначально. И со своими объектами могу деоать всё что угодно.
 
 - есть идеи как убрать эти права? если нет - смотрите шпаргалку
 
testdb=# revoke CREATE on SCHEMA public FROM public;<BR>
REVOKE<BR>
testdb=# revoke all on DATABASE testdb FROM public;<BR>
REVOKE<BR>
testdb=#<BR>
 
 
 - если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
 
 Подглядел про дефалтовые права. Ну и скрипы ревоука паблик скопипастил.
 
 - теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
 
 testdb=> create table t3(c1 integer); insert into t2 values (2);<BR>
ERROR:  permission denied for schema public<BR>
INSERT 0 1<BR>

 
 - расскажите что получилось и почему 
 
 Таблицу в паблик мы вставить теперь не имеем права. А строка вставилась, потому что владельцев таблицы t2 всё ещё остались мы:<BR>
         List of relations<BR>
 Schema | Name | Type  |  Owner<BR>
--------+------+-------+----------<BR>
 public | t2   | table | testread<BR>
(1 row)<BR>

