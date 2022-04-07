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
 - Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
 - Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
