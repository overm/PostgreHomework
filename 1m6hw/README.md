 - создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a

создал gp1-1m6hw
 - поставьте на нее PostgreSQL 14 через sudo apt
 
 sudo apt update<BR>
sudo apt install postgresql postgresql-contrib<BR>
sudo pg_ctlcluster 12 main start
 - проверьте что кластер запущен через sudo -u postgres pg_lsclusters

pavel@gp1-1m6hw:~$ sudo -u postgres pg_lsclusters<BR>
Ver Cluster Port Status Owner    Data directory              Log file<BR>
12  main    5432 online postgres /var/lib/postgresql/12/main /var/log/postgresql/postgresql-12-main.log<BR>

 - зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым postgres=# create table test(c1 text); postgres=# insert into test values('1'); \q
 
 pavel@gp1-1m6hw:~$ sudo -u postgres psql<BR>
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))<BR>
Type "help" for help.<BR>
<BR>
postgres=# create table t1(a int);<BR>
CREATE TABLE<BR>
postgres=# insert into t1 values(1),(2);<BR>
INSERT 0 2<BR>
postgres=# select * from t1;<BR>
 a<BR>
---<BR>
 1<BR>
 2<BR>
(2 rows)<BR>

 - остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop
 
 pavel@gp1-1m6hw:~$ sudo pg_ctlcluster 12 main stop

 - создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
 
 создал ssd disk1
 - добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
 ![image](https://user-images.githubusercontent.com/16693077/159765960-228a3ec7-8c97-4bf8-b141-17a0f7ac2454.png)

 - проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

 pavel@gp1-1m6hw:~$ lsblk<BR>
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT<BR>
vda    252:0    0  15G  0 disk<BR>
├─vda1 252:1    0   1M  0 part<BR>
└─vda2 252:2    0  15G  0 part /<BR>
vdb    252:16   0  20G  0 disk<BR>

 
pavel@gp1-1m6hw:~$ sudo mkfs.ext4 /dev/vdb<BR>
mke2fs 1.45.5 (07-Jan-2020)<BR>
Creating filesystem with 5242880 4k blocks and 1310720 inodes<BR>
Filesystem UUID: b3ea10b4-83de-4165-9e83-3b6fa85f542f<BR>
Superblock backups stored on blocks:<BR>
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,<BR>
        4096000<BR>
<BR>
Allocating group tables: done<BR>
Writing inode tables: done<BR>
Creating journal (32768 blocks): done<BR>
Writing superblocks and filesystem accounting information: done<BR>
<BR>
cd /mnt/sudo<BR>
sudo mkdir data<BR>
 sudo mount /dev/vdb /mnt/data/<BR>
 sudo chown postgres:postgres /mnt/data/<BR>
 
 отредактировал sudo nano /etc/fstab 
 - перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

 pavel@gp1-1m6hw:~$ mount | grep mnt<BR>
/dev/vdb on /mnt/data type ext4 (rw,relatime)

 
 - сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
 
 выше сразу сделал
 - перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data
 
 sudo mv /var/lib/postgresql/12 /mnt/data
 
 - попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
 
 pavel@gp1-1m6hw:/var$ sudo -u postgres pg_ctlcluster 12 main start<BR>
Error: /var/lib/postgresql/12/main is not accessible or does not exist

 
 - напишите получилось или нет и почему
 
 По старому пути нету базы
 - задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
 
 pavel@gp1-1m6hw:/etc/postgresql/12/main$ cat  postgresql.conf | grep data_directory<BR>
data_directory = '/var/lib/postgresql/12/main'          # use data in another directory<BR>
 стало<BR>
 pavel@gp1-1m6hw:/etc/postgresql/12/main$ cat  postgresql.conf | grep data_directory<BR>
data_directory = '/mnt/data/12/main'           # use data in another directory<BR>


 - напишите что и почему поменяли
 
 путь к каталогу с данными
 - попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
 
 sudo -u postgres pg_ctlcluster 12 main start
 - напишите получилось или нет и почему

  pavel@gp1-1m6hw:/etc/postgresql/12/main$ sudo -u postgres pg_ctlcluster 12 main start<BR>
Warning: the cluster will not be running as a systemd service. Consider using systemctl:<BR>
  sudo systemctl start postgresql@12-main<BR>
Removed stale pid file.<BR>
 Получилось, потому что СУБД нашла путь к базе<BR>
 - зайдите через через psql и проверьте содержимое ранее созданной таблицы
 
pavel@gp1-1m6hw:/etc/postgresql/12/main$ sudo -u postgres psql<BR>
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))<BR>
Type "help" for help.<BR>
<BR>
postgres=# select * from t1;<BR>
 a<BR>
---<BR>
 1<BR>
 2<BR>
(2 rows)<BR>


 
 - задание со звездочкой *: не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

 Создал gp2-1m6hw<BR>
 Оставновил первую виртуалку<BR>
 Переподсоединил диск ко второй<BR>
 Установил PG sudo apt install postgresql postgresql-contrib<BR>
  sudo mkdir /mnt/data<BR>
 sudo mount /dev/vdb /mnt/data/<BR>
 sudo chown postgres:postgres /mnt/data/<BR>
 sudo pg_ctlcluster 12 main stop<BR>
 sudo nano /etc/postgresql/12/main/postgresql.conf<BR>
 pavel@gp2-1m6hw:~$ cat  /etc/postgresql/12/main/postgresql.conf | grep data_directory<BR>
data_directory = '/mnt/data/12/main'            # use data in another directory<BR>
 sudo -u postgres pg_ctlcluster 12 main start<BR>
 sudo -u postgres psql<BR>
 pavel@gp2-1m6hw:~$ sudo -u postgres psql<BR>
psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))<BR>
Type "help" for help.<BR>
<BR>
postgres=# select * from t1;<BR>
 a<BR>
---<BR>
 1<BR>
 2<BR>
(2 rows)<BR>
