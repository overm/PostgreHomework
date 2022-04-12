 - развернуть виртуальную машину любым удобным способом

Стало интересно, что можно выжать из облачной виртуалки. Сразу сделал максимально доступный ссд, квота которого осталась доступной,в моём случае 160ГБ для максималного ИО. А память и процессоры на период настройки зажал, но как настрою, разверну на 32 ГБ и 16 ядер.
 - поставить на неё PostgreSQL 14 из пакетов собираемых postgres.org

``` console
pavel@hw12:~$ sudo -u postgres psql -c "SELECT version();"
                                                               version
--------------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0, 64-bit
(1 row)
```

 - настроить кластер PostgreSQL 14 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины

Я сперва на слабеньком кластере поставил все утилиты. Сделал пробный прогон, развернул в целевые ресурсы. Об этом подробно в последнем пункте.

 - нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench) или через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)

Поставил рекомендованные утилиты:
``` console
pavel@hw12:~/sysbench-tpcc$ sysbench --version
sysbench 1.0.20
```

``` console
pavel@hw12:~/sysbench-tpcc$ ./tpcc.lua --version
sysbench 1.0.20
```

 - написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему

 
Итак на на маленьком КТС с 2 ядрами 2 ГБ с дефалтовым настройками попытался заполнить данными:
```
./tpcc.lua --pgsql-user=postgres --pgsql-db=sbtest --time=120 --threads=5 --report-interval=1 --tables=10 --scale=100  --pgsql-password=*** --db-driver=pgsql prepare
sysbench 1.0.20 (using bundled LuaJIT 2.1.0-beta2)

Initializing worker threads...
...
```

Не дождался, всё было очень медленно. Срубил, остановил машину, нарастил ресурсов. Сделал 50 сессий. Дождался.

 - Попытка 1

Настройки по умолчанию. Перед основным прогоном делаю минутный прогон, чтобы заполнить все кэши. Основная команда тестирования:
```
./tpcc.lua --pgsql-user=postgres --pgsql-db=sbtest --time=300 --threads=30 --report-interval=1 --tables=10 --scale=100  --pgsql-password=dsvcjndsucidsu8sd --db-driver=pgsql run
```

``` console
SQL statistics:
    queries performed:
        read:                            278914
        write:                           289077
        other:                           43198
        total:                           611189
    transactions:                        21515  (71.30 per sec.)
    queries:                             611189 (2025.51 per sec.)
    ignored errors:                      149    (0.49 per sec.)
    reconnects:                          0      (0.00 per sec.)
```

ресурсы пустые
```
top - 18:37:42 up 32 min,  2 users,  load average: 22.70, 9.81, 5.35
Tasks: 398 total,   1 running, 397 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.2 us,  0.4 sy,  0.0 ni, 29.0 id, 69.2 wa,  0.0 hi,  0.1 si,  0.1 st
MiB Mem :  64314.6 total,  50782.5 free,    474.7 used,  13057.4 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.  62985.1 avail Mem
```
Пока 71.3 tps

- Попытка 2

Берём базовые настройки pgtune
```
max_connections = 30
shared_buffers = 16GB
effective_cache_size = 48GB
maintenance_work_mem = 2GB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 69905kB
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 32
max_parallel_workers_per_gather = 4
max_parallel_workers = 32
max_parallel_maintenance_workers = 4
```

Смотрим результаты:
```
SQL statistics:
    queries performed:
        read:                            348468
        write:                           361291
        other:                           54022
        total:                           763781
    transactions:                        26894  (89.28 per sec.)
    queries:                             763781 (2535.42 per sec.)
    ignored errors:                      203    (0.67 per sec.)
    reconnects:                          0      (0.00 per sec.)
```


Увы ресурсы всё равно простаивают
```
top - 18:48:05 up 43 min,  2 users,  load average: 23.43, 17.08, 11.70
Tasks: 401 total,   1 running, 400 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  0.6 sy,  0.0 ni, 39.1 id, 59.1 wa,  0.0 hi,  0.1 si,  0.0 st
MiB Mem :  64314.6 total,  38895.3 free,    802.0 used,  24617.3 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.  56986.4 avail Mem
```

Результат немного подрос 89.28tps


Я посмотрел на монитор и, судя по всему, нашёл узкое горлышко:
![image](https://user-images.githubusercontent.com/16693077/163033463-d55ef695-4810-4437-b051-4db0102891ea.png)
На 160ГБ SSD доступно порядка 65МБ/с. То есть скоррее всего мы упрёлись в потолок по IO.


 - Попытка 3

Пойдём по хардкору, в качестве хрранилища сделаем RAM-disk, на виртуалке никак не обойдём ограничение на бесплатном аккаунте по IO.
