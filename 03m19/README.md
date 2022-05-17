Вернулся снова своему инстансу GreenPlum, настроил подключениес DataGrip. Попробую домашку выполнить на нём.
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

 - Создать индекс к какой-либо из таблиц вашей БД
 - Прислать текстом результат команды explain, в которой используется данный индекс
 - Реализовать индекс для полнотекстового поиска
 - Реализовать индекс на часть таблицы или индекс на поле с функцией
 - Создать индекс на несколько полей
 - Написать комментарии к каждому из индексов
 - Описать что и как делали и с какими проблемами столкнулись 
