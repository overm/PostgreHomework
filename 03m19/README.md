Вернулся снова к своему инстансу GreenPlum, настроил подключениес DataGrip. Попробую домашку выполнить на нём.

h1 Вариант 1
==============
![image](https://user-images.githubusercontent.com/16693077/168903455-ac8226ae-c07a-413d-9853-1c91111397a9.png)
Создадим и наполним таблицу
``` sql
create table my
(id integer not null,
descip varchar
)
with (appendoptimized=true)
distributed by (id)
;

insert into my
SELECT generate_series(1,100000) AS id, md5(random()::text) AS descr;

analyse my;

select count(*) from my;
```
![image](https://user-images.githubusercontent.com/16693077/168904701-5e9d0eef-d008-4561-8d38-47bbc38d386f.png)

 - Создать индекс к какой-либо из таблиц вашей БД
``` sql
CREATE  INDEX my_ind ON my (id);

explain
select * from my where id = 223;
```
 - Прислать текстом результат команды explain, в которой используется данный индекс
``` console
Gather Motion 1:1  (slice1; segments: 1)  (cost=0.00..387.96 rows=1 width=37)
  ->  Bitmap Heap Scan on my  (cost=0.00..387.96 rows=1 width=37)
        Recheck Cond: (id = 223)
        ->  Bitmap Index Scan on my_ind  (cost=0.00..0.00 rows=0 width=0)
              Index Cond: (id = 223)
Optimizer: Pivotal Optimizer (GPORCA)
```
 - Реализовать индекс для полнотекстового поиска

Реализовал таблицу my2, но текстовым типом tsvector
``` sql
create table my2
(id integer not null,
descip tsvector
)
with (appendoptimized=true)
distributed by (id)
;
insert into my2
SELECT generate_series(1,100000) AS id, md5(random()::text)::tsvector AS descr;

CREATE INDEX my_ind_gist ON my2 USING gist(descip);
```
 - Реализовать индекс на часть таблицы или индекс на поле с функцией
``` sql
CREATE  INDEX my_ind_func ON my (lower(descip));
```
 - Создать индекс на несколько полей
``` sql
CREATE  INDEX my_ind_mc ON my (id, descip);
```
 - Написать комментарии к каждому из индексов

Первый самый базовый b-tree. Второй для полнотекстового поиска. Третий для поиска по результату функции. Четвёртый многоколоночный b-tree.
 - Описать что и как делали и с какими проблемами столкнулись 
Не было никаких проблем. Разве что думал, что для gist можно использовать тип text, но оказалось нужен специальный тип.



h1 Вариант 2
==============
Есть три таблицы с частично пересекающимися айдишниками, см последний раздел
 - Реализовать прямое соединение двух или более таблиц
![image](https://user-images.githubusercontent.com/16693077/168920362-a42130b2-f6e3-46b2-a9d4-34fa038cccf6.png)

 - Реализовать левостороннее (или правостороннее) соединение двух или более таблиц
![image](https://user-images.githubusercontent.com/16693077/168920503-0294b879-e4e1-4452-8ca0-dabe26d34d78.png)

 - Реализовать кросс соединение двух или более таблиц
 ![image](https://user-images.githubusercontent.com/16693077/168920685-2cce26a9-a792-439e-94b8-bc392e5455c1.png)

 - Реализовать полное соединение двух или более таблиц
![image](https://user-images.githubusercontent.com/16693077/168921114-8e8f6862-f86a-4c67-8736-6505f47dc700.png)

- Реализовать запрос, в котором будут использованы разные типы соединений
![image](https://user-images.githubusercontent.com/16693077/168921699-125308be-2639-4e69-ae05-3988722840db.png)

 - Сделать комментарии на каждый запрос
Написал перед каждой картинкой
 - К работе приложить структуру таблиц, для которых выполнялись соединения
``` sql
create table my1
(id integer not null,
descip varchar
)
with (appendoptimized=true)
distributed by (id)
;

insert into my1
SELECT generate_series(1,10) AS id, md5(random()::text) AS descr;


create table my2
(id integer not null,
descip varchar
)
with (appendoptimized=true)
distributed by (id)
;

insert into my2
SELECT generate_series(3,13) AS id, md5(random()::text) AS descr;


create table my3
(id integer not null,
descip varchar
)
with (appendoptimized=true)
distributed by (id)
;

insert into my3
SELECT generate_series(6,16) AS id, md5(random()::text) AS descr;
```
