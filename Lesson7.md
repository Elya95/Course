# Блокировки

1. Настройка, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд:
otus=# show deadlock_timeout;
 deadlock_timeout 
 1s
(1 row)

otus=# ALTER SYSTEM SET deadlock_timeout TO 200;
ALTER SYSTEM
otus=# select pg_reload_conf();
 pg_reload_conf 
 t
(1 row)

otus=# show deadlock_timeout;
 deadlock_timeout 
 200ms
(1 row)

*Информация о созданных блокировках:*
```
2023-08-06 07:12:01.541 UTC [5777] postgres@postgres ERROR:  deadlock detected
2023-08-06 07:12:01.541 UTC [5777] postgres@postgres DETAIL:  Process 5777 waits for ShareLock on transaction 516; blocked by process 5787.
Process 5787 waits for ShareLock on transaction 515; blocked by process 5777.
   
```
2.  Во время попытки обновить одну и ту же строку, тремя сессиями, первая сессия блокирует вторую, а та в свою очередь - третью. 
Первая сессия:
 txid_current | pg_backend_pid 
--------------+----------------
       331815 |           6127
Вторая сессия: 
 txid_current | pg_backend_pid 
--------------+----------------
       331818 |           8412
Третья сессия:
 txid_current | pg_backend_pid 
--------------+----------------
       331819 |           8424

**Текущие блокировки:**
- *Каждая сессия держит эксклюзивные (exclusive lock) блокировки;*
Первая сессия:
otus=*# SELECT * FROM locks_v WHERE pid = 6127; 
 pid  |   locktype    |  lockid  |       mode       | granted 
------+---------------+----------+------------------+---------
 6127 | transactionid | 331815   | ExclusiveLock    | t
 Вторая:
 otus=*# SELECT * FROM locks_v WHERE pid = 8412; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
 8412 | transactionid | 331818     | ExclusiveLock    | t
 Третья:
 otus=*# SELECT * FROM locks_v WHERE pid = 8424; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
  8424 | transactionid | 331819     | ExclusiveLock    | t
- *Первая сессия захватила эксклюзивную блокировку строки (RowExclusiveLock) для самой строки, что пытаются обновить и первая и другие сессии.*
otus=*# SELECT * FROM locks_v WHERE pid = 6127; 
 pid  |   locktype    |  lockid  |       mode       | granted 
------+---------------+----------+------------------+---------
  6127 | relation      | accounts | RowExclusiveLock | t
 - *Другие сессии так же захватили эксклюзивную блокировку строки, которую пытаются все обновить:*
otus=*# SELECT * FROM locks_v WHERE pid = 8412; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
 8412 | relation      | accounts   | RowExclusiveLock | t
 
otus=*# SELECT * FROM locks_v WHERE pid = 8424; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
 8424 | relation      | accounts   | RowExclusiveLock | t
- *Вторая и третья сессия так же повесили эксклюзивную блокировку (ExclusiveLock) на саму запись:*
otus=*# SELECT * FROM locks_v WHERE pid = 8412; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
 8412 | tuple         | accounts:1 | ExclusiveLock    | t

otus=*# SELECT * FROM locks_v WHERE pid = 8424; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
 8424 | tuple         | accounts:1 | ExclusiveLock    | f

- *Блокировка share lock вызвана тем, что мы пытаемся обновить ту же строку что и в первой сессии, у которого уже захвачен row exclusive lock:*
otus=*# SELECT * FROM locks_v WHERE pid = 8412; 
 pid  |   locktype    |   lockid   |       mode       | granted 
------+---------------+------------+------------------+---------
 8412 | transactionid | 331815     | ShareLock        | f

3. С помощью журнала сообщений, можно посмотреть какими процессами вызваны блокировки, посмотреть запрос, который вызвал блокировку. Какой тип блокировки возник при данной ситуации, а так же номера транзакции вызывающие блокировку. 
4. Да могут, так как в любом случае, два сеанса пересекутся при обновлении данных и возникнет взаимоблокировка. 
