 - Настройте выполнение контрольной точки раз в 30 секунд.
``` console
pavel@m2hw8:~$ cat /etc/postgresql/13/main/postgresql.conf | grep checkpoint_ti
checkpoint_timeout = 30s                # range 30s-1d
```
 - 10 минут c помощью утилиты pgbench подавайте нагрузку.
 - Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

Сперва я думал, что просто посмотрю каталог /var/lib/postgresql/13/main/pg_wal, но они по кучу раз перезаписались и ничего осмысленного я не увидел:
``` console
postgres@m2hw8:~/13/main/pg_wal$ ls -la
total 213004
drwx------  3 postgres postgres     4096 Apr  3 08:44 .
drwx------ 19 postgres postgres     4096 Apr  3 08:31 ..
-rw-------  1 postgres postgres 16777216 Apr  3 08:43 0000000100000000000000E6
-rw-------  1 postgres postgres 16777216 Apr  3 08:44 0000000100000000000000E7
-rw-------  1 postgres postgres 16777216 Apr  3 08:40 0000000100000000000000E8
-rw-------  1 postgres postgres 16777216 Apr  3 08:39 0000000100000000000000E9
-rw-------  1 postgres postgres 16777216 Apr  3 08:40 0000000100000000000000EA
-rw-------  1 postgres postgres 16777216 Apr  3 08:40 0000000100000000000000EB
-rw-------  1 postgres postgres 16777216 Apr  3 08:41 0000000100000000000000EC
-rw-------  1 postgres postgres 16777216 Apr  3 08:41 0000000100000000000000ED
-rw-------  1 postgres postgres 16777216 Apr  3 08:42 0000000100000000000000EE
-rw-------  1 postgres postgres 16777216 Apr  3 08:42 0000000100000000000000EF
-rw-------  1 postgres postgres 16777216 Apr  3 08:42 0000000100000000000000F0
-rw-------  1 postgres postgres 16777216 Apr  3 08:43 0000000100000000000000F1
-rw-------  1 postgres postgres 16777216 Apr  3 08:43 0000000100000000000000F2
drwx------  2 postgres postgres     4096 Mar 29 18:05 archive_status
```

Запишем текущий LSN
``` sql
postgres=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 0/EAAADE10
(1 row)
```
После отработки gpstat
``` sql
postgres=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 1/5739B98
(1 row)


postgres=# SELECT pg_size_pretty('1/5739B98'::pg_lsn - '0/EAAADE10'::pg_lsn);
 pg_size_pretty
----------------
 429 MB
(1 row)
```

На одну точку с учётом таймаута должно приходится около 21,5 мб, но из списка файлов мы видим из размер ровно по 16мб. Не могу это объяснить.

 - Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?

```
postgres=# SELECT * FROM pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 206
checkpoints_req       | 0
checkpoint_write_time | 9189305
checkpoint_sync_time  | 4196
buffers_checkpoint    | 186064
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 52541
buffers_backend_fsync | 0
buffers_alloc         | 56318
stats_reset           | 2022-03-29 18:05:48.876161+00
```
Всё ок

 - Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

В синхронном режиме
``` console
postgres@m2hw8:~/13/main$ pgbench -c8 -P 60 -T 60 -U postgres postgres
starting vacuum...end.
progress: 60.0 s, 592.4 tps, lat 13.499 ms stddev 13.064
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 35552
latency average = 13.498 ms
latency stddev = 13.063 ms
tps = 592.460887 (including connections establishing)
tps = 592.479295 (excluding connections establishing)
```
В асинхронном
``` console
pavel@m2hw8:/var/lib/postgresql/13$ cat /etc/postgresql/13/main/postgresql.conf | grep synchron
# - Asynchronous Behavior -
synchronous_commit = off                # synchronization level;

postgres@m2hw8:~/13/main$ pgbench -c8 -P 60 -T 60 -U postgres postgres
starting vacuum...end.
progress: 60.0 s, 6364.4 tps, lat 1.254 ms stddev 1.215
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 60 s
number of transactions actually processed: 381871
latency average = 1.254 ms
latency stddev = 1.215 ms
tps = 6363.981635 (including connections establishing)
tps = 6364.189565 (excluding connections establishing)
```

Разница более чем в 10 раз. Круто!

 - Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

Включим контрольные суммы
``` console
postgres@m2hw8:~/13/main$ /usr/lib/postgresql/13/bin/pg_checksums --enable -D "/var/lib/postgresql/13/main"
Checksum operation completed
Files scanned:  931
Blocks scanned: 7559
pg_checksums: syncing data directory
pg_checksums: updating control file
Checksums enabled in cluster
```

``` sql
postgres=# select * from qwe;
  a  |                                                  b
-----+-----------------------------------------------------------------------------------------------------
 111 | asdddddddddddddddddddasssssssssssssssssddddddddddddddddddddxcccccccccccccccccccccccxcssssssssssssss
 121 | asdddddddddddddddddddasssssssssssssssssddddddddddddddddddddxcccccccccccccccccccccccxcssssssssssssss
 131 | asdddddddddddddddddddasssssssssssssssssddddddddddddddddddddxcccccccccccccccccccccccxcssssssssssssss
(3 rows)

postgres=# SELECT pg_relation_filepath('qwe');
 pg_relation_filepath
----------------------
 base/13448/18296
(1 row)
```

Измнил файл после остановки кластера:
``` sql
postgres=# select * from qwe;
WARNING:  page verification failed, calculated checksum 11021 but expected 34682
ERROR:  invalid page in block 0 of relation base/13448/18296

postgres=# SET ignore_checksum_failure = on;
SET
postgres=# select * from qwe;
WARNING:  page verification failed, calculated checksum 11021 but expected 34682
  a  |                                                  b
-----+-----------------------------------------------------------------------------------------------------
 111 | asdddddddddddddddddddasssssssssssssssssdddddddddgdgddddddddxcccccccccccccccccccccccxcssssssssssssss
 121 | asdddddddddddddddddddasssssssssssssssssddddddddddddddddddddxcccccccccccccccccccccccxcssssssssssssss
 131 | asdddddddddddddddddddasssssssssssssssssddddddddddddddddddddxcccccccccccccccccccccccxcssssssssssssss
(3 rows)
```
