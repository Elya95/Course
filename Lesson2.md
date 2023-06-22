# Установка PostgreSQL

## Запуск контейнера сервера Postgres
1) **При запуске команды:
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
Возникла ошибка:**
> Error starting userland proxy: listen tcp4 0.0.0.0:5432: bind: address already in use.

* Смотрим какой процесс задействует порт 5432:
 root@Master1:/var/lib/postgresql# sudo ss -lptn 'sport = :5432'
State                  Recv-Q                 Send-Q                                 Local Address:Port                                 Peer Address:Port                Process                
LISTEN                 0                      244                                        127.0.0.1:5432                                      0.0.0.0:*                    users:(("postgres",pid=10171,fd=5))
* root@Master1:/var/lib/postgresql# sudo kill 10171

***Заново запускаем команду: 
root@Master1:/var/lib/postgresql# sudo docker run --rm --name pgdocker -e POSTGRES_PASSWORD=02091995 -e POSTGRES_USER=postgres  -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres
Она выполняется успешно.***

2. **Созданный контейнер, мы не подключили к нашей созданной сети pg-net. 
На данный момент наш контейнер подключен к сети bridge, по умолчанию:
root@Master1:/var/lib/postgresql# docker network inspect bridge**
> "Containers": {
"345dba9421c802cee84c68375abec91829d796fe5136633fe89675d462eac250": {
"Name": "pgdocker",
"EndpointID": "7bfb9b6d04ed8a6dd3d00e1234517854ab7bee5e57f5118877c6aa1c57045fbc",
"MacAddress": "02:42:ac:11:00:02",
"IPv4Address": "172.17.0.2/16",
"IPv6Address": ""
}
* Создаем docker-сеть: 
sudo docker network create pg-net
* Удалим созданный контейнер:
root@Master1:/var/lib/postgresql# docker rm -f pgdocker
pgdocker  
* Заново создаем его:
root@Master1:/var/lib/postgresql# sudo docker run --rm --name pgdocker --network pg-net -e POSTGRES_PASSWORD=02091995 -e POSTGRES_USER=postgres  -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres
* Проверяем:
root@Master1:/var/lib/postgresql# docker network inspect pg-net
> "Containers": {
           "31750ff884632fff56b77506fd45f29d28edaa0789fd2520bc5f3123aaca04bf": {
                "Name": "pgdocker",
                "EndpointID": "85f08c34ef452287110e60960ae02cb4970aa2f7253299d634fc5e84ac6a196b",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }

**Вывод:
Во время выполнения домашнего задания, проблемы с которыми встречалась, описаны мною выше. Данные действительно сохранились даже после удаления контейнера, так как была примонтирован внешний каталог**