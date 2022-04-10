 - Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

``` console
pavel@m2hw8:~$ cat  /etc/postgresql/13/main/postgresql.conf | grep 'lock_lo\|dead'
log_lock_waits = 1                      # log lock waits >= deadlock_timeout
deadlock_timeout = 200ms
```
блокировка
``` console
Первая сессия
postgres=# begin;
BEGIN
postgres=*# select * from qwe where a=1 for update;
 a |  b
---+-----
 1 | qwe
(1 row)

postgres=*# commit
postgres-*# ;
COMMIT



Вторая сессия
postgres=# select * from qwe where a=1 for update;
 a |  b
---+-----
 1 | qwe
(1 row)



Лог
2022-04-07 18:53:31.677 UTC [1508] postgres@postgres LOG:  process 1508 still waiting for ShareLock on transaction 6881319 after 200.129 ms
2022-04-07 18:53:31.677 UTC [1508] postgres@postgres DETAIL:  Process holding the lock: 1571. Wait queue: 1508.
2022-04-07 18:53:31.677 UTC [1508] postgres@postgres CONTEXT:  while locking tuple (0,1) in relation "qwe"
2022-04-07 18:53:31.677 UTC [1508] postgres@postgres STATEMENT:  select * from qwe where a=1 for update;
2022-04-07 18:53:39.339 UTC [1508] postgres@postgres LOG:  process 1508 acquired ShareLock on transaction 6881319 after 7862.137 ms
2022-04-07 18:53:39.339 UTC [1508] postgres@postgres CONTEXT:  while locking tuple (0,1) in relation "qwe"
2022-04-07 18:53:39.339 UTC [1508] postgres@postgres STATEMENT:  select * from qwe where a=1 for update;
```

 - Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

Табличка
``` console
postgres=# select * from qwe;
 a |  b
---+-----
 1 | qwe
 2 | asd
 3 | zxc
(3 rows)
```

Первая транзакция:
``` console
postgres=# BEGIN;
BEGIN
postgres=*# SELECT pg_backend_pid();
 pg_backend_pid
----------------
            952
(1 row)

postgres=*# update qwe set b='ert' where a=1;
UPDATE 1
```

Вторая транзакция:
``` console
postgres=# BEGIN;
BEGIN
postgres=*# SELECT pg_backend_pid();
 pg_backend_pid
----------------
            956
(1 row)

postgres=*# update qwe set b='dfg' where a=1;
```

Третья транзакция:
``` console
postgres=# BEGIN;
BEGIN
postgres=*# SELECT pg_backend_pid();
 pg_backend_pid
----------------
            960
(1 row)

postgres=*# update qwe set b='cvb' where a=1;
```

Посмотрим блокировки:
``` console
postgres=# SELECT pid, locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
FROM pg_locks order by pid;
 pid  |   locktype    | relation | virtxid |   xid   |       mode       | granted
------+---------------+----------+---------+---------+------------------+---------
  952 | relation      | qwe      |         |         | RowExclusiveLock | t
  952 | transactionid |          |         | 6881324 | ExclusiveLock    | t
  952 | virtualxid    |          | 3/40    |         | ExclusiveLock    | t
  956 | tuple         | qwe      |         |         | ExclusiveLock    | t
  956 | relation      | qwe      |         |         | RowExclusiveLock | t
  956 | virtualxid    |          | 4/5     |         | ExclusiveLock    | t
  956 | transactionid |          |         | 6881324 | ShareLock        | f
  956 | transactionid |          |         | 6881325 | ExclusiveLock    | t
  960 | tuple         | qwe      |         |         | ExclusiveLock    | f
  960 | transactionid |          |         | 6881326 | ExclusiveLock    | t
  960 | virtualxid    |          | 5/5     |         | ExclusiveLock    | t
  960 | relation      | qwe      |         |         | RowExclusiveLock | t
 1229 | virtualxid    |          | 6/262   |         | ExclusiveLock    | t
 1229 | relation      | pg_locks |         |         | AccessShareLock  | t
```
Видим, что первая pid 952 получил блокировку на qwe 6881324. Вторая pid 956 создал встала в очередь на 6881324 (не получив) и создала блокировку 6881325 и turple. Третья 960 создала блокировку 6881326 и встала в очередь в turple.

 - Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

Сперва в каждой сессии сделал апдейт по одной строке, затем в каждой уже по второй:
``` console
postgres=# BEGIN;
BEGIN
postgres=*# SELECT pg_backend_pid();
 pg_backend_pid
----------------
            952
(1 row)

postgres=*# update qwe set b='ert' where a=1;
UPDATE 1
postgres=*# update qwe set b='ert' where a=2;
````

``` console
BEGIN
postgres=*# SELECT pg_backend_pid();
 pg_backend_pid
----------------
            956
(1 row)

postgres=*# update qwe set b='dfg' where a=2;
UPDATE 1
postgres=*# update qwe set b='dfg' where a=3;
UPDATE 1
```

``` console
postgres=# BEGIN;
BEGIN
postgres=*# SELECT pg_backend_pid();
 pg_backend_pid
----------------
            960
(1 row)

postgres=*# update qwe set b='cvb' where a=3;
UPDATE 1
postgres=*# update qwe set b='cvb' where a=1;
ERROR:  deadlock detected
DETAIL:  Process 960 waits for ShareLock on transaction 6881327; blocked by process 952.
Process 952 waits for ShareLock on transaction 6881328; blocked by process 956.
Process 956 waits for ShareLock on transaction 6881329; blocked by process 960.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "qwe"
```

Теперь смотрим лог
``` console
2022-04-09 19:27:41.130 UTC [952] postgres@postgres LOG:  process 952 still waiting for ShareLock on transaction 6881328 after 200.153 ms
2022-04-09 19:27:41.130 UTC [952] postgres@postgres DETAIL:  Process holding the lock: 956. Wait queue: 952.
2022-04-09 19:27:41.130 UTC [952] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "qwe"
2022-04-09 19:27:41.130 UTC [952] postgres@postgres STATEMENT:  update qwe set b='ert' where a=2;
2022-04-09 19:27:48.605 UTC [956] postgres@postgres LOG:  process 956 still waiting for ShareLock on transaction 6881329 after 200.179 ms
2022-04-09 19:27:48.605 UTC [956] postgres@postgres DETAIL:  Process holding the lock: 960. Wait queue: 956.
2022-04-09 19:27:48.605 UTC [956] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "qwe"
2022-04-09 19:27:48.605 UTC [956] postgres@postgres STATEMENT:  update qwe set b='dfg' where a=3;
2022-04-09 19:27:56.983 UTC [960] postgres@postgres LOG:  process 960 detected deadlock while waiting for ShareLock on transaction 6881327 after 200.160 ms
2022-04-09 19:27:56.983 UTC [960] postgres@postgres DETAIL:  Process holding the lock: 952. Wait queue: .
2022-04-09 19:27:56.983 UTC [960] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "qwe"
2022-04-09 19:27:56.983 UTC [960] postgres@postgres STATEMENT:  update qwe set b='cvb' where a=1;
2022-04-09 19:27:56.983 UTC [960] postgres@postgres ERROR:  deadlock detected
2022-04-09 19:27:56.983 UTC [960] postgres@postgres DETAIL:  Process 960 waits for ShareLock on transaction 6881327; blocked by process 952.
        Process 952 waits for ShareLock on transaction 6881328; blocked by process 956.
        Process 956 waits for ShareLock on transaction 6881329; blocked by process 960.
        Process 960: update qwe set b='cvb' where a=1;
        Process 952: update qwe set b='ert' where a=2;
        Process 956: update qwe set b='dfg' where a=3;
2022-04-09 19:27:56.983 UTC [960] postgres@postgres HINT:  See server log for query details.
2022-04-09 19:27:56.983 UTC [960] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "qwe"
2022-04-09 19:27:56.983 UTC [960] postgres@postgres STATEMENT:  update qwe set b='cvb' where a=1;
2022-04-09 19:27:56.983 UTC [956] postgres@postgres LOG:  process 956 acquired ShareLock on transaction 6881329 after 8578.938 ms
2022-04-09 19:27:56.983 UTC [956] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "qwe"
2022-04-09 19:27:56.983 UTC [956] postgres@postgres STATEMENT:  update qwe set b='dfg' where a=3;
```
Ну примерно всё то же самое: 952 стала ждать 956. Ту стала ждать 960, а потом потом 960 попыталась встать в очередь в 952 со всемивытекающими дедлоками.

 - Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

Я конечно был в шоке от того, что postgre накладывает построчную блокировку, даже если знает, что будет апдейтить всю таблицу.В процессе исследования нашёл команду lock, будем использовать её для областей, где могут попадаться конкурентные апдейты и другие операции..

