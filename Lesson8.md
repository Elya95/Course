# Настройка PostgreSQL

1. Нагрузка при помощи pgbench с первоначальными параметрами кластера:
*******************
 max_connections = '100';
 shared_buffers = '128MB';
 effective_cache_size = '4GB';
 maintenance_work_mem = '64MB';
 checkpoint_completion_target = '0.9';
 wal_buffers = '4MB';
 default_statistics_target = '100';
 random_page_cost = '4';
 effective_io_concurrency = '1';
 work_mem = '4MB';
 min_wal_size = '80MB';
 max_wal_size = '1GB';
 
_______________________________ 
 tps = 694.984865 (without initial connection time)
 _______________________________
 
2.  Далее я использовала ресурс https://pgtune.leopard.in.ua/
```
# DB Version: 15
# OS Type: linux
# DB Type: web
# Total Memory (RAM): 10 GB
# CPUs num: 2
# Connections num: 100
# Data Storage: hdd

max_connections = 100
shared_buffers = 2560MB
effective_cache_size = 7680MB
maintenance_work_mem = 640MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 13107kB
min_wal_size = 1GB
max_wal_size = 4GB
```
В 20 раз увеличили параметр shared_buffers
Практически в два раза увеличили параметр effective_cache_size.
В 10 раз увеличили параметр maintenance_work_mem.
В 4 раза увеличили параметр  wal_buffers.
В 2 раза увеличили параметр effective_io_concurrency.
В 3 раза увеличили параметр work_mem. 
В 12 раз увеличили параметр  min_wal_size и в 4 раза параметр max_wal_size.
***************
tps = 856.146932 (without initial connection time)
**************
3.  Далее если мы поднимем в три раза от прошлого значения параметр  max_connections, в три раза уменьшим значение work_mem, значение tps сильно не изменится. 
```
-- DB Version: 15
-- OS Type: linux
-- DB Type: web
-- Total Memory (RAM): 10 GB
-- CPUs num: 2
-- Connections num: 300
-- Data Storage: hdd

ALTER SYSTEM SET
 max_connections = '300';
ALTER SYSTEM SET
 shared_buffers = '2560MB';
ALTER SYSTEM SET
 effective_cache_size = '7680MB';
ALTER SYSTEM SET
 maintenance_work_mem = '640MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '4';
ALTER SYSTEM SET
 effective_io_concurrency = '2';
ALTER SYSTEM SET
 work_mem = '4369kB';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';
```
***************
tps = 869.331127 (without initial connection time)
***************

4. При настройке для OLTP системы, уменьшения количества max_connections в 10 раз, work_mem увеличивается в 10 раз. Значение tps мы добились самого наименьшего. 
```
-- DB Version: 15
-- OS Type: linux
-- DB Type: oltp
-- Total Memory (RAM): 10 GB
-- CPUs num: 2
-- Connections num: 30
-- Data Storage: hdd

ALTER SYSTEM SET
 max_connections = '30';
ALTER SYSTEM SET
 shared_buffers = '2560MB';
ALTER SYSTEM SET
 effective_cache_size = '7680MB';
ALTER SYSTEM SET
 maintenance_work_mem = '640MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '4';
ALTER SYSTEM SET
 effective_io_concurrency = '2';
ALTER SYSTEM SET
 work_mem = '43690kB';
ALTER SYSTEM SET
 min_wal_size = '2GB';
ALTER SYSTEM SET
 max_wal_size = '8GB';
```
****************
**tps = 553.747561 (without initial connection time)**
****************
5. С параметрами выше, были сделаны еще следующие изменения параметров:
- wal_level=minimal (Удаляет все журналы, кроме информации, необходимой для восстановления после сбоя или немедленного завершения работы. Уровень `minimal`генерирует наименьший объем WAL. Он не регистрирует информацию о строках для постоянных отношений в транзакциях, которые их создают или перезаписывают. Это может значительно ускорить операции)
- synchronous_commit=off (Нет ожидания локального сброса WAL на диск, увеличивает производительность, но не обеспечивает гаранти. сохранности каждой транзакции)
- checkpoint_timeout=180s (Чем больше интервал между контрольными точками, тем меньше объём записи в журнал WAL и тем меньше объем обмена с диском)
****************
**tps = 1924.224269 (without initial connection time)**
********************
Число транзакций в секунду значительно увеличилось. 