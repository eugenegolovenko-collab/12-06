# Домашнее задание к занятию "`Репликация и масштабирование`" - `Евгений Головенко`

---

### Задание 1

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Решение 1

Конфигурация Master:

`dockerfile_master`
``` docker
FROM mysql:8.0
COPY ./master.cnf /etc/mysql/conf.d/my.cnf
COPY ./master.sql /docker-entrypoint-initdb.d/master.sql
ENV MYSQL_ROOT_PASSWORD=rootpass
CMD ["mysqld"]
```

`master.cnf`
```
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog_format=ROW
```
`master.sql`
```
CREATE USER 'repl'@'%' IDENTIFIED BY 'slavepass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```
Конфигурация Slave:

`dockerfile_slave`
```
FROM mysql:8.0
COPY ./slave.cnf /etc/mysql/conf.d/my.cnf
COPY ./slave.sql /docker-entrypoint-initdb.d/start.sql
ENV MYSQL_ROOT_PASSWORD=rootpass
CMD ["mysqld"]
```
`slave.cnf`
```
[mysqld]
server-id = 2
read-only = 1
super-read-only = 0
relay-log = relay-bin
```
`slave.sql`
```
CHANGE REPLICATION SOURCE TO
SOURCE_HOST='mysql_master',
SOURCE_USER='repl',
SOURCE_PASSWORD='slavepass',
SOURCE_SSL=1;
START REPLICA;
```

Старт контейнеров:

<img width="1082" height="602" alt="Conts" src="https://github.com/user-attachments/assets/b13f0c75-591e-47f1-b861-0cee8e209aad" />

Проверка сети:
```
PS E:\IT_Courses\Netologia\Data_Bases\12-06\Repl> docker network inspect replication
[
    {
        "Name": "replication",
        "Id": "42b7c053f9f94442469785d4b389fe7b9936491d3d4008568cb1b24d372456a8",
        "Created": "2026-01-20T18:42:18.538218991Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "IPRange": "",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Options": {
            "com.docker.network.enable_ipv4": "true",
            "com.docker.network.enable_ipv6": "false"
        },
            "com.docker.network.enable_ipv6": "false"
        },
            "com.docker.network.enable_ipv6": "false"
        },
            "com.docker.network.enable_ipv6": "false"                                                 
        },
        "Labels": {},                                                                               
        "Containers": {
            "c5af6192f7cea4e6be1efb98a8f7f48a9684ed7871c4497da187b22ec67a02a6": {
                "Name": "mysql_slave",
                "EndpointID": "17085203bf2dabddd3f3ade3cc84de81c706f89c6f82abb0f0691082f000773e",  
                "MacAddress": "36:92:6c:fe:61:18",
                "IPv4Address": "172.18.0.3/16",                                                       
                "IPv6Address": ""
            },                                                                                      
            "ed1d9ee10e0314448b1534510f27984e1456d1099d898dc7880883d7e33b57e1": {
                "Name": "mysql_master",
                "EndpointID": "710d550aaafa087cc92c1b58070722b6dcb10eb80424d387e31ea2f2123e02a7",  
                "MacAddress": "7e:35:e2:1f:ab:8f",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Status": {
            "IPAM": {
                "Subnets": {
                    "172.18.0.0/16": {
                        "IPsInUse": 5,
                        "DynamicIPsAvailable": 65531
                    }
                }
            }
        }
    }
]
```

Состояния и режимы работы:

<img width="899" height="590" alt="master_status" src="https://github.com/user-attachments/assets/2faf234e-8bbd-4775-a5ff-c43d563f5f3e" />

<img width="1402" height="588" alt="slave_status" src="https://github.com/user-attachments/assets/ca3d6e87-80c2-4b6d-8830-b19ae566f8d8" />

Проверка работы репликации:

<img width="783" height="594" alt="master_rec" src="https://github.com/user-attachments/assets/40886e52-5d33-4fe2-b15d-4bc78ed6a242" />

<img width="771" height="589" alt="slave_rep" src="https://github.com/user-attachments/assets/1d7e59a2-09e0-4bda-b3a6-2225ae8f86bb" />

---

### Задание 2

Разработайте план для выполнения горизонтального и вертикального шаринга базы данных. База данных состоит из трёх таблиц: 

- пользователи, 
- книги, 
- магазины (столбцы произвольно). 

Опишите принципы построения системы и их разграничение или разбивку между базами данных.

*Пришлите блоксхему, где и что будет располагаться. Опишите, в каких режимах будут работать сервера.* 

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 3* 

Опишите основные преимущества использования масштабирования методами:

- активный master-сервер и пассивный репликационный slave-сервер; 
- master-сервер и несколько slave-серверов;
- активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD);
- SAN-кластер.

*Дайте ответ в свободной форме.*

---

### Задание 4*

Выполните настройку выбранных методов шардинга из задания 2.

*Пришлите конфиг Docker и SQL скрипт с командами для базы данных*.

---
