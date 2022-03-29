 - создать GCE инстанс типа e2-medium и standard disk 10GB

Не знаю, что такое e2-edium, поэтому взял на яндексе машину помощнее 8ядер, 16ГБ
om@play-box:~$ ssh pavel@51.250.105.45<BR>
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-96-generic x86_64)<BR>
<BR>
 * Documentation:  https://help.ubuntu.com<BR>
 * Management:     https://landscape.canonical.com<BR>
 * Support:        https://ubuntu.com/advantage<BR>
Last login: Tue Mar 29 17:54:59 2022 from 212.46.15.3<BR>
pavel@m2hw8:~$<BR>

 - установить на него PostgreSQL 13 с дефолтными настройками
 
 pavel@m2hw8:~$ sudo pg_ctlcluster 13 main status<BR>
pg_ctl: server is running (PID: 3200)<BR>
/usr/lib/postgresql/13/bin/postgres "-D" "/var/lib/postgresql/13/main" "-c" "config_file=/etc/postgresql/13/main/postgresql.conf"

 
 - применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
 
 И тутя открыл параметры и понял, что погорячился  с конфигурацией. Попробуем пока так.<BR>
 pavel@m2hw8:~$ cat /etc/postgresql/13/main/postgresql.conf | grep 'max_connections shared_buffers\|effective_cache_size\|maintenance_work_mem\|checkpoint_completion_target\|wal_buffers\|default_statistics_target\|random_page_cost\|effective_io_concurrency\|work_mem\|min_wal_size\|max_wal_size'<BR>
work_mem = 6553kB                               # min 64kB<BR>
#hash_mem_multiplier = 1.0              # 1-1000.0 multiplier on hash table work_mem<BR>
maintenance_work_mem = 512MB            # min 1MB<BR>
#autovacuum_work_mem = -1               # min 1MB, or -1 to use maintenance_work_mem<BR>
#logical_decoding_work_mem = 64MB       # min 64kB<BR>
effective_io_concurrency = 2            # 1-1000; 0 disables prefetching<BR>
wal_buffers = 16MB                      # min 32kB, -1 sets based on shared_buffers<BR>
max_wal_size = 16GB<BR>
min_wal_size = 4GB<BR>
checkpoint_completion_target = 0.9      # checkpoint target duration, 0.0 - 1.0<BR>
random_page_cost = 4.0                  # same scale as above<BR>
effective_cache_size = 3GB<BR>
default_statistics_target = 500 # range 1-10000<BR>

 
 - выполнить pgbench -i postgres
 
 postgres@m2hw8:/home/pavel$ pgbench -i postgres<BR>
dropping old tables...<BR>
NOTICE:  table "pgbench_accounts" does not exist, skipping<BR>
NOTICE:  table "pgbench_branches" does not exist, skipping<BR>
NOTICE:  table "pgbench_history" does not exist, skipping<BR>
NOTICE:  table "pgbench_tellers" does not exist, skipping<BR>
creating tables...<BR>
generating data (client-side)...<BR>
100000 of 100000 tuples (100%) done (elapsed 0.05 s, remaining 0.00 s)<BR>
vacuuming...<BR>
creating primary keys...<BR>
done in 2.01 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 1.68 s, vacuum 0.06 s, primary keys 0.26 s).<BR>
 
 - запустить pgbench -c8 -P 60 -T 3600 -U postgres postgres
 - дать отработать до конца
 - зафиксировать среднее значение tps в последней ⅙ части работы
 
 Excel сказал 581,29<BR>
 ![image](https://user-images.githubusercontent.com/16693077/160692170-388dbb7d-d4b3-41bc-bcc2-2385df68ee50.png)

 
 - а дальше настроить autovacuum максимально эффективно
 
 Как я понимаю задумку, то колебания вызваны bloat'ом таблиц и возрастанием производительности после отработки vacuum'а. То есть если сделать запуск вакуума по каждому чиху, то ситуация будет ровной. Для этих целей попробуем настройки из лекции.<BR>
 ![image](https://user-images.githubusercontent.com/16693077/160693264-9d50ecd2-81c3-407f-87e4-3c9cdaadfe24.png)

 
 - так чтобы получить максимально ровное значение tps на горизонте часа
 
 Не уверен, что это идеальный результат. Я уменьшил количество строк срабатывания до 10 и результат стал вращаться вокруг цифры 600, е опускаясь ниже 550tps, исключая начало.
![image](https://user-images.githubusercontent.com/16693077/160709738-4fb50d59-1d3a-4ab5-85ec-a84a0e1f880b.png)


 
