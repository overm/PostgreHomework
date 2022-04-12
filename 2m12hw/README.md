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
        read:                            59248
        write:                           61726
        other:                           9196
        total:                           130170
    transactions:                        4552   <br>(73.34 per sec.)<br>
    queries:                             130170 (2097.23 per sec.)
    ignored errors:                      34     (0.55 per sec.)
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
