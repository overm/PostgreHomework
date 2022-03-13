- сделать в GCE инстанс с Ubuntu 20.04

Создал виртуалку на Yandex Cloud - Ubuntu 20, при создании указал прошлый ключ<BR>
 yc vpc addresses list<BR>
+----------------------+------+-----------------+----------+------+<BR>
|          ID          | NAME |     ADDRESS     | RESERVED | USED |<BR>
+----------------------+------+-----------------+----------+------+<BR>
| e9bolavkp86jv9nuhe3n |      | 178.154.229.112 | false    | true |<BR>
+----------------------+------+-----------------+----------+------+<BR>
 Подсключился:<BR>
  ![image](https://user-images.githubusercontent.com/16693077/158060193-12646254-3c74-4cc5-8eef-1f8e0593e9e1.png)

- поставить на нем Docker Engine
 
 Сделал по инструкции https://docs.docker.com/engine/install/ubuntu/<BR>
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose
- сделать каталог /var/lib/postgres
 
 sudo mkdir /var/lib/postgres
- развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
 
 Скопировал docker-compose.yml, немного поменял, запустил:<BR>
 sudo docker-compose up -d
- развернуть контейнер с клиентом postgres
 
 Нашёл контейнер с клиентов, 12 - но работает:<BR>
 sudo docker run -it --rm jbergknoff/postgresql-client postgresql://postgres:*******@178.154.229.112:5432/stage 
- подключиться из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

 Unable to find image 'jbergknoff/postgresql-client:latest' locally<BR>
latest: Pulling from jbergknoff/postgresql-client<BR>
df20fa9351a1: Pull complete<BR>
6b7bc9b9c3f1: Pull complete<BR>
Digest: sha256:45e175ebb700cfd46e23a610477c3576550055ef40c394e663623946a5eced39<BR>
Status: Downloaded newer image for jbergknoff/postgresql-client:latest<BR>
psql (12.3, server 14.2 (Debian 14.2-1.pgdg110+1))<BR>
WARNING: psql major version 12, server major version 14.<BR>
         Some psql features might not work.<BR>
Type "help" for help.<BR>

stage=# create table t1 (a int);<BR>
CREATE TABLE<BR>
stage=# insert into t1 values(1);<BR>
INSERT 0 1<BR>
stage=# insert into t1 values(2);<BR>
INSERT 0 1<BR>

- подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP

Тут бы конечно SSL бы не помешал, ну да ладно.<BR>
sudo apt install postgresql-client-common postgresql-client-12<BR>
psql -h 178.154.229.112 -d stage -U postgres<BR>
Password for user postgres:<BR>
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1), server 14.2 (Debian 14.2-1.pgdg110+1))<BR>
WARNING: psql major version 12, server major version 14.<BR>
         Some psql features might not work.<BR>
Type "help" for help.<BR>
<BR>
stage=# select * from t1;<BR>
 a<BR>
---<BR>
 1<BR>
 2<BR>
(2 rows)<BR>

- удалить контейнер с сервером

pavel@postgre:~$ sudo docker container ls<BR>
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES<BR>
8108fcc664e1   postgres:14   "docker-entrypoint.s…"   32 minutes ago   Up 32 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pavel_pg_db_1<BR>
pavel@postgre:~$ sudo docker rm pavel_pg_db_1<BR>
Error response from daemon: You cannot remove a running container 8108fcc664e121d3ca061bd0ddf99b44a5b759f91708c6d39cd0643fa3f970bb. Stop the container before attempting removal or force remove<BR>
pavel@postgre:~$ sudo docker stop pavel_pg_db_1<BR>
pavel_pg_db_1<BR>
pavel@postgre:~$ sudo docker rm pavel_pg_db_1<BR>
pavel_pg_db_1<BR>

- создать его заново
 
pavel@postgre:~$ sudo docker-compose up -d<BR>
Creating pavel_pg_db_1 ... done
- подключится снова из контейнера с клиентом к контейнеру с сервером
 
sudo docker run -it --rm jbergknoff/postgresql-client postgresql://postgres:njsdkjnfskjdnfASD123@178.154.229.112:5432/stage<BR>
psql (12.3, server 14.2 (Debian 14.2-1.pgdg110+1))<BR>
WARNING: psql major version 12, server major version 14.<BR>
         Some psql features might not work.<BR>
Type "help" for help.<BR>
<BR>
stage=# <BR>


- проверить, что данные остались на месте

 stage=# select * from t1;<BR>
 a<BR>
---<BR>
 1<BR>
 2<BR>
(2 rows)<BR>

- оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

 Да особо никаких проблем не было, на яндексе даже с портом возиться не пришлось.
