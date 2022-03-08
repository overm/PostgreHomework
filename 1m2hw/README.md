# SQL и реляционные СУБД. Введение в PostgreSQL 
 - создать новый проект в Google Cloud Platform, например postgres2022-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально на уровне GCP)

В GCP не удалось привязать карту. Создал проект postgres2022-19870531 в Яндекс-облаке.
 - дать возможность доступа к этому проекту пользователю ifti@yandex.ru с ролью Project Editor

Добавил пользователя  ifti@yandex.ru с ролью viwer.
 - далее создать инстанс виртуальной машины Compute Engine с дефолтными параметрами
Создал через веб-морду sshtest
Установил и заргистрировал терминал yc. Запросил на будущее NAT как сервис, если не надут - сделаю отдельную машину, чтобы не плодить внишние айпишники.

Создал ключ
om@play-box:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/om/.ssh/id_rsa): sshtest
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in sshtest
Your public key has been saved in sshtest.pub
The key fingerprint is:
SHA256:2Tx81Da7BbDq1eu8l6EZckIr9wUCFLKHF+cncIhdYq8 om@play-box.chilikin.pro
The key's randomart image is:
+---[RSA 3072]----+
|       .+B++.    |
|       .=+O  +   |
|       o o.+o.=  |
|        o=.=o+ + |
|        SEB = + .|
|         o O o * |
|          + = B o|
|             * ..|
|              +o |
+----[SHA256]-----+

Добавил к себе:
om@play-box:~$ ssh-add sshtest
Identity added: sshtest (om@play-box.chilikin.pro)


 - добавить свой ssh ключ в GCE metadata
При создании виртуалки указал в окошке
 - зайти удаленным ssh (первая сессия), не забывайте про ssh-add
Смотрим список виртуалок:
om@play-box:~$ yc vpc network list
+----------------------+---------+
|          ID          |  NAME   |
+----------------------+---------+
| enpkmjl6cgerir1bmqb2 | default |
+----------------------+---------+

 - поставить PostgreSQL
 - зайти вторым ssh (вторая сессия)
 - запустить везде psql из под пользователя postgres
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





