import_kladr_postgresql
=======================

Загрузка КЛАДР в PostgreSQL

Конвертация КЛАДР в формат sqlite.

    Скачиваем кладр с официального сайта
    wget www.gnivc.ru/html/gnivcsoft/KLADR/Base.7z
    Устанавливаем архиватор 7z
    sudo yum install p7zip
    Распаковываем архив
    7za e Base.7z
    Устанавливаем sqlite
    sudo yum install sqlite
    Устанавливаем sqlite3-dbf
    sudo yum install sqlite3-dbf
    Запускаем sqlite3
    sqlite3 my_kladr.db
    В sqlite загружаем модуль libspatialite
    .load libspatialite.so.2
    Импорт данных из КЛАДР в sqlite
    CREATE VIRTUAL TABLE virt_street_tbl USING VirtualDbf('/home/developer/kladr/STREET.DBF', 'CP866');
    CREATE VIRTUAL TABLE virt_socrbase_tbl USING VirtualDbf('/home/developer/kladr/SOCRBASE.DBF', 'CP866');
    CREATE VIRTUAL TABLE virt_kladr_tbl USING VirtualDbf('/home/developer/kladr/KLADR.DBF', 'CP866');
    CREATE VIRTUAL TABLE virt_flat_tbl USING VirtualDbf('/home/developer/kladr/FLAT.DBF', 'CP866');
    CREATE VIRTUAL TABLE virt_doma_tbl USING VirtualDbf('/home/developer/kladr/DOMA.DBF', 'CP866');
    CREATE VIRTUAL TABLE virt_altnames_tbl USING VirtualDbf('/home/developer/kladr/ALTNAMES.DBF', 'CP866');

    create table street_tbl as select * from virt_street_tbl;
    create table socrbase_tbl as select * from virt_socrbase_tbl;
    create table kladr_tbl as select * from virt_kladr_tbl;
    create table flat_tbl as select * from virt_flat_tbl;
    create table doma_tbl as select * from virt_doma_tbl;
    create table altnames_tbl as select * from virt_altnames_tbl;

    drop table virt_street_tbl;
    drop table virt_socrbase_tbl;
    drop table virt_kladr_tbl;
    drop table virt_flat_tbl;
    drop table virt_doma_tbl;
    drop table virt_altnames_tbl;
    Выходим из sqlite
    .exit

Резлуьтат: файл my_kladr.db содержит КЛАДР в формате sqlite.

Подключение файла my_kladr.db к базе данных PostgreSQL

    Скачиваем модуль sqlite_fdw
    wget https://github.com/gleu/sqlite_fdw/archive/master.zip
    Устанавливаем unzip
    sudo yum install unzip
    Распаковываем архив
    unzip master
    Заходим в каталог sqlite_fdw-master
    cd sqlite_fdw-master
    Устанавливаем модуль
    sudo PATH=/usr/pgsql-9.3/bin/:$PATH make USE_PGXS=1 install
    Входим в систему из под пользователя postgres
    sudo su - postgres
    Входим в postgresql
    psql
    Выбираем базу данных
    \c YourDatabase
    Создаем расширение
    CREATE EXTENSION sqlite_fdw;
    Создаем сервер
    CREATE SERVER sqlite_kladr_server
        FOREIGN DATA WRAPPER sqlite_fdw
        OPTIONS (database 'path_to_my_kladr.db');
    Создаем схему
    create schema kladr;
    Создаем внешние таблицы
    CREATE FOREIGN TABLE kladr.street_tbl(
      id bigint,
      name varchar,
      type varchar,
      code varchar,
      c2 varchar,
      c3 varchar,
      c4 varchar,
      c5 varchar)
        SERVER sqlite_kladr_server
         OPTIONS (table 'street_tbl');

    CREATE FOREIGN TABLE kladr.socrbase_tbl(
      id bigint,
      id1 bigint,
      short_name varchar,
      full_name varchar,
      id3 bigint)
        SERVER sqlite_kladr_server
         OPTIONS (table 'socrbase_tbl');

    CREATE FOREIGN TABLE kladr.kladr_tbl(
      id bigint,
      name varchar,
      type varchar,
      code varchar,
      c4 varchar,
      c5 varchar,
      c6 varchar,
      c7 bigint)
        SERVER sqlite_kladr_server
         OPTIONS (table 'kladr_tbl');

    CREATE FOREIGN TABLE kladr.doma_tbl(
      id bigint,
      house varchar,
      c1 varchar,
      c2 varchar,
      c3 varchar,
      c4 varchar,
      c5 varchar,
      c6 varchar,
      c7 varchar)
        SERVER sqlite_kladr_server
         OPTIONS (table 'doma_tbl');

    CREATE FOREIGN TABLE kladr.altnames_tbl(
      id bigint,
      code1 varchar,
      code2 varchar,
      c1 varchar)
        SERVER sqlite_kladr_server
         OPTIONS (table 'altnames_tbl');
    Проверяем работу
    select * from kladr.kladr_tbl limit 10;

Результат: Кладр подключен к базе данных PostgreSQL.
