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

Итак на на маленьком КТС с 2 ядрами 2 ГБ с дефалтовым настройками:
```
./tpcc.lua --pgsql-user=postgres --pgsql-db=sbtest --time=120 --threads=5 --report-interval=1 --tables=10 --scale=100  --pgsql-password=*** --db-driver=pgsql prepare
sysbench 1.0.20 (using bundled LuaJIT 2.1.0-beta2)

Initializing worker threads...
...
```
