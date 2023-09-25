# Резервное копирование и восстановление

1. БД otus, schema - elka, user - elka.
2. Таблица - test. Заполнила таблицу автосгенерированными 100 записями.
3.  Каталог для бэкапов под postgres:
'backup'.
4. Логический бэкап первой таблицы test:
\copy test to '/tmp/backup/1.sql'
5. Вторая таблица - test1, восстановили туда данные из первой таблицы:
\copy test1 from '/tmp/backup/1.sql';
otus=> select * from test1 limit 10;
 i  
----
  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
(10 rows)
6. Бэкап в кастомном сжатом формате двух таблиц: 
pg_dump -d otus -p 5432 -E LATIN1 -t elka.test -t elka.test1  -Fc > /tmp/backup/4.gz
7. Восстановим в новую БД только вторую таблицу
pg_restore -d otus1 -p 5432 -t test1 -Fc /tmp/backup/4.gz