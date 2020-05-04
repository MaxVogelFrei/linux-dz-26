# MySQL InnoDB Cluster
## Задание
развернуть InnoDB кластер в docker  

в качестве ДЗ принимает репозиторий с docker-compose  
который по кнопке разворачивает кластер и выдает порт наружу  
## Решение
в ВМ mysql устанавливается docker, pip, mysql-shell  
стенд разворачивается при провижене машины выполнением  
```bash
docker-compose -f /vagrant/docker-compose.yml up -d
```
с отключенным selinux для доступа к скрипту настройки кластера  

docker-compose.yml запускает 5 контейнеров:  
3 сервера MySQL  
1 роутер MySQL  
1 контейнер для запуска js скрипта настройки кластера  

с роутера проброшен порт TCP/6446  
## Проверка
после авторизации в ВМ mysql выполняю "mysqlsh" и подключаюсь к localhost:6446  
MySQL root пароль OtusLinux2020  

```bash
[root@mysql ~]# mysqlsh
MySQL Shell 8.0.20

Copyright (c) 2016, 2020, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.
 MySQL  JS > \connect localhost:6446
Creating a session to 'root@localhost:6446'
Please provide the password for 'root@localhost:6446': *************
Fetching schema names for autocompletion... Press ^C to stop.
Your MySQL connection id is 434
Server version: 8.0.20 MySQL Community Server - GPL
No default schema selected; type \use <schema> to set one.
 MySQL  localhost:6446 ssl  JS > dba.getCluster().status()
WARNING: No cluster change operations can be executed because the installed metadata version 1.0.1 is lower than the version required by Shell which is version 2.0.0. Upgrade the metadata to remove this restriction. See \? dba.upgradeMetadata for additional details.
{
    "clusterName": "VogelFrei", 
    "defaultReplicaSet": {
        "name": "default", 
        "primary": "mysql-server-1:3306", 
        "ssl": "REQUIRED", 
        "status": "OK", 
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.", 
        "topology": {
            "mysql-server-1:3306": {
                "address": "mysql-server-1:3306", 
                "mode": "n/a", 
                "readReplicas": {}, 
                "role": "HA", 
                "shellConnectError": "MySQL Error 2005 (HY000): Unknown MySQL server host 'mysql-server-1' (2)", 
                "status": "ONLINE", 
                "version": "8.0.20"
            }, 
            "mysql-server-2:3306": {
                "address": "mysql-server-2:3306", 
                "mode": "n/a", 
                "readReplicas": {}, 
                "role": "HA", 
                "shellConnectError": "MySQL Error 2005 (HY000): Unknown MySQL server host 'mysql-server-2' (2)", 
                "status": "ONLINE", 
                "version": "8.0.20"
            }, 
            "mysql-server-3:3306": {
                "address": "mysql-server-3:3306", 
                "mode": "n/a", 
                "readReplicas": {}, 
                "role": "HA", 
                "shellConnectError": "MySQL Error 2005 (HY000): Unknown MySQL server host 'mysql-server-3' (2)", 
                "status": "ONLINE", 
                "version": "8.0.20"
            }
        }, 
        "topologyMode": "Single-Primary"
    }, 
    "groupInformationSourceMember": "87abb1c6cc95:3306"
}
 MySQL  localhost:6446 ssl  JS > 
```
