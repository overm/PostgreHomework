# SQL и реляционные СУБД. Введение в PostgreSQL 
 - создать новый проект в Google Cloud Platform, например postgres2022-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально на уровне GCP)

В GCP не удалось привязать карту. Создал проект postgres2022-19870531 в Яндекс-облаке.
 - дать возможность доступа к этому проекту пользователю ifti@yandex.ru с ролью Project Editor

Добавил пользователя  ifti@yandex.ru с ролью viwer.
 - далее создать инстанс виртуальной машины Compute Engine с дефолтными параметрами

Создал через веб-морду sshtest<br>
Установил и заргистрировал терминал yc. Запросил на будущее NAT как сервис, если не надут - сделаю отдельную машину, чтобы не плодить внишние айпишники.<br>

Создал ключ<br>
om@play-box:~$ ssh-keygen -t rsa<br>
Generating public/private rsa key pair.<br>
Enter file in which to save the key (/home/om/.ssh/id_rsa): sshtest<br>
Enter passphrase (empty for no passphrase):<br>
Enter same passphrase again:<br>
Your identification has been saved in sshtest<br>
Your public key has been saved in sshtest.pub<br>
The key fingerprint is:<br>
SHA256:2Tx81Da7BbDq1eu8l6EZckIr9wUCFLKHF+cncIhdYq8 om@play-box.chilikin.pro<br>
The key's randomart image is:<br>
+---[RSA 3072]----+<br>
|       .+B++.    |<br>
|       .=+O  +   |<br>
|       o o.+o.=  |<br>
|        o=.=o+ + |<br>
|        SEB = + .|<br>
|         o O o * |<br>
|          + = B o|<br>
|             * ..|<br>
|              +o |<br>
+----[SHA256]-----+<br>

Добавил к себе:<br>
om@play-box:$ ssh-add sshtest<br>
Identity added: sshtest (om@play-box.chilikin.pro)
 - добавить свой ssh ключ в GCE metadata

При создании виртуалки указал в окошке
 - зайти удаленным ssh (первая сессия), не забывайте про ssh-add

Смотрим список виртуалок:<br>
om@play-box:~$ yc vpc addresses list<br>
+----------------------+------+----------------+----------+------+<br>
|          ID          | NAME |    ADDRESS     | RESERVED | USED |<br>
+----------------------+------+----------------+----------+------+<br>
| e9bq5f12m9qviqifer8a |      | 178.154.231.59 | false    | true |<br>
+----------------------+------+----------------+----------+------+<br>

om@play-box:$ ssh pavel@178.154.231.59<br>
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-96-generic x86_64)<br>
<br>
 Documentation:  https://help.ubuntu.com<br>
 Management:     https://landscape.canonical.com<br>
 Support:        https://ubuntu.com/advantage<br>
pavel@sshtest:~$

 - поставить PostgreSQL

sudo apt update<br>
sudo apt install postgresql postgresql-contrib<br>
pg_ctlcluster 12 main start
 - зайти вторым ssh (вторая сессия)

sudo -u postgres psql
 - запустить везде psql из под пользователя 
 ![image](https://user-images.githubusercontent.com/16693077/157284091-e70e1894-6d2b-4252-bdd1-21446a6f275d.png)

 sudo -u postgres psql
 - выключить auto commit
 - сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
 - посмотреть текущий уровень изоляции: show transaction isolation level
 - начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
 - в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
 - сделать select * from persons во второй сессии
 - видите ли вы новую запись и если да то почему?
 - завершить первую транзакцию - commit;
 - сделать select * from persons во второй сессии
 - видите ли вы новую запись и если да то почему?
 - завершите транзакцию во второй сессии
 - начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
 - в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
 - сделать select * from persons во второй сессии
 - видите ли вы новую запись и если да то почему?
 - завершить первую транзакцию - commit;
 - сделать select * from persons во второй сессии
 - видите ли вы новую запись и если да то почему?
 - завершить вторую транзакцию
 - сделать select * from persons во второй сессии
 - видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите





