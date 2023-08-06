# Журналы

**Настройка контрольной точки раз в 30 секунд:**
1. ALTER SYSTEM SET checkpoint_timeout = '30s';
SELECT pg_reload_conf();
2. После подачи нагрузки с помощью pgbench, за 5 минут было сгенерировано 218 MB. Около 21 MB на одну контрольную точку. 
- Позиция в журнале перед подачей нагрузки:
otus=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn 
 0/D1770B40
(1 row)
- Сброс статистики:
otus=# SELECT pg_stat_reset_shared('bgwriter');
 pg_stat_reset_shared 
(1 row)
- После подачи нагрузки в 5 минут, позиция в журнале: 
otus=# SELECT pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn 
 0/DF1ED260
(1 row)
- **Объем журнальных файлов за 5 минут:**
otus=# SELECT pg_size_pretty('0/DF1ED260'::pg_lsn - '0/D1770B40'::pg_lsn);
 pg_size_pretty 
 218 MB
(1 row)
3. otus=# SELECT checkpoints_timed, checkpoints_req FROM pg_stat_bgwriter;
 checkpoints_timed | checkpoints_req 
                13              |               0
(1 row)
* checkpoints_timed — контрольные точки по расписанию (checkpoint_timeout);
* checkpoints_req — контрольные точки по требованию (max_wal_size);
Все контрольные точки выполнялись точно по расписанию. Это говорит о том, что в среднем объем журнальных записей за контрольную точку не превосходит установленного предела. 
4. Разница в два раза, из-за более эффективного сбрасывания на диск. 
5. Включение контрольной суммы гарантирует целостность файлов и выдаст сообщение, если файл был поврежден. Чтобы избежать такую ошибку еще раз, можно удалить поврежденные строки или применить настройку зануляющую поврежденные строки и выполнить полный вакуум.
SET zero_damaged_pages = on;
vacuum full test1; --наша таблица, где были повреждены строки. 
