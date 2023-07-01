# Логический уровень PostgreSQL

**Почему под пользователем testread не получилось увидеть созданную таблицу t1, хоть и были предоставлены права. 
Возникла ошибка:**
> postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# \c - testread
Password for user testread: 
You are now connected to database "testdb" as user "testread".
testdb=> select * from t1;
ERROR:  permission denied for table t1

**Права на все таблицы схемы testnm были предоставлены данному пользователю. Но, данная таблица создалась в схеме public. А на использование и просмотр объектов этой таблицы, права предоставлены не были.**
> testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres

**Так произошло, потому что в search_path ="$user", public , а так как $USER схемы нет, то таблица по умолчанию создалась в public:**

>.testdb=> show search_path;  
   search_path   
  "$user", public
(1 row)

**Как вариант, после создания базы данных testdb, чтобы все объекты создавались в новой схеме testnm, можно было выполнить команду:**
>ALTER DATABASE testdb SET search_path = testnm, public;

**Либо создавать таблицу, явно указывая схему:**
>CREATE TABLE testnm.t1(c1 integer);

**Даже после того, как мы пересоздали таблицу с явным указанием схемы, под пользователем postgres, testread так и не увидел пересозданную таблицу. Потому что права были предоставлены только для существующих таблиц, а не новых. Чтобы это изменить необходимо выполнить команду:**
>\c testdb postgres; 
   ALTER default privileges in SCHEMA testnm grant SELECT on      TABLES to readonly;
   grant SELECT on all TABLEs in SCHEMA testnm TO readonly;

***После этого пользователь testread увидит содержимое таблицы t1.***

____________________________________________________________
**Более того под пользователем testread, мы смогли создать таблицу. Потому что у каждого пользователя есть право создавать объекты в схеме postgres. Права работать с этой схемой есть у каждого пользователя. Если у него есть право подключаться к базе. Все пользователи состоят в роли public. Для того, чтобы исключить возможность создания объектов в данной схеме, достаточно сделать вот эти 2 команды:**
> \c testdb postgres; 
REVOKE CREATE on SCHEMA public FROM public; 
REVOKE ALL on DATABASE testdb FROM public; 

**После данных команд, создавать таблицы под testread мы уже не можем:**
>testdb=> create table t2 (c1 int);
ERROR:  permission denied for schema public
LINE 1: create table t2 (c1 int);



<img src="https://media.istockphoto.com/id/1316444316/ru/%D0%B2%D0%B5%D0%BA%D1%82%D0%BE%D1%80%D0%BD%D0%B0%D1%8F/%D0%BA%D1%80%D0%B0%D1%81%D0%BD%D1%8B%D0%B9-%D0%B2%D0%B5%D0%BA%D1%82%D0%BE%D1%80-%D0%B8%D0%BB%D0%BB%D1%8E%D1%81%D1%82%D1%80%D0%B0%D1%86%D0%B8%D0%B8-%D0%B1%D0%B0%D0%BD%D0%BD%D0%B5%D1%80-%D0%B2%D0%B0%D0%B6%D0%BD%D0%BE-%D1%81-%D0%B2%D0%BE%D1%81%D0%BA%D0%BB%D0%B8%D1%86%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%BC-%D0%B7%D0%BD%D0%B0%D0%BA%D0%BE%D0%BC.jpg?s=1024x1024&w=is&k=20&c=Xf5mvZVMPSlIhTKdwbqeChQ84tTbV3KOHuHKsfDVnY0=" />