# Физический уровень PostgreSQL

**После переноса содержимого data_directory (postgres) в  /mnt/data - кластер не запустился. 
Возникла ошибка:**
> Error: /var/lib/postgresql/15/main is not accessible or does not exist

**Для того чтобы кластер запустился, нужно указать новое размещение файлов данных, после их переноса. В /etc/postgresql/15/main/postgresql.conf правим параметр data_directory = '/mnt/data/15/main'**

***Итог:
postgres@Replica1:/etc/postgresql/15/main$ pg_ctlcluster 15 main status
pg_ctl: server is running (PID: 1907)
/usr/lib/postgresql/15/bin/postgres "-D" "/mnt/data/15/main" "-c" "config_file=/etc/postgresql/15/main/postgresql.conf"
Сервер успешно запущен. Таблица, которую мы создавали ранее, существует даже после переноса данных в другое место***

<img src="https://resizer.mail.ru/p/7855035a-ddb8-5d6a-a17d-39f18345dabd/AQAC9aqFv8PttNCJw7KTFRetHUairgdxdei2ETmLiH-wsgZ147tE5vP62JrjNQeTwycz7WSpD8JEAlg5MVvudEoe87c.webp" />


 

