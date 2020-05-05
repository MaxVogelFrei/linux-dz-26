# MySQL InnoDB Cluster
## Задание
развернуть InnoDB кластер в docker  

в качестве ДЗ принимает репозиторий с docker-compose  
который по кнопке разворачивает кластер и выдает порт наружу  
## Решение
в ВМ mysql устанавливается docker, pip, docker-compose и mysql-shell  
```bash
yum install docker python-pip python-devel gcc -y
pip install --upgrade pip
pip install docker-compose
yum install https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm -y
yum install mysql-shell -y
```
стенд разворачивается при провижене машины выполнением  
```bash
docker-compose -f /vagrant/docker-compose.yml up -d
```
с отключенным selinux для доступа к скрипту настройки кластера  

###docker-compose.yml запускает 5 контейнеров:  
#### 3 сервера MySQL
```yaml
  mysql-server-1:
    env_file:
      - mysql-server.env
    image: mysql/mysql-server:latest
    ports:
      - "3301:3306"
    command: ["mysqld","--server_id=1","--binlog_checksum=NONE","--gtid_mode=ON","--enforce_gtid_consistency=ON","--log_bin","--log_slave_updates=ON","--master_info_repository=TABLE","--relay_log_info_repository=TABLE","--transaction_write_set_extraction=XXHASH64","--user=mysql","--skip-host-cache","--skip-name-resolve", "--default_authentication_plugin=mysql_native_password"]
```
в контейнеры передается пароль для root@%
```bash 
MYSQL_ROOT_PASSWORD=OtusLinux2020
MYSQL_ROOT_HOST=%
```
#### роутер MySQL
```yaml
  mysql-router:
    env_file:
      - mysql-router.env
    image: mysql/mysql-router:latest
    ports:
      - "6446:6446"
    depends_on:
      - mysql-server-1
      - mysql-server-2
      - mysql-server-3
      - mysql-shell
    restart: on-failure
```
С роутера проброшен порт TCP/6446  
Роутеру передаются данные для подключения к серверу mysql  
```bash
MYSQL_USER=root
MYSQL_HOST=mysql-server-1
MYSQL_PORT=3306
MYSQL_PASSWORD=OtusLinux2020
MYSQL_INNODB_NUM_MEMBERS=3
```
#### контейнер для запуска js скрипта настройки кластера  
```yaml
  mysql-shell:
    env_file:
      - mysql-shell.env
    image: neumayer/mysql-shell-batch
    volumes:
        - ./scripts/:/scripts/
    depends_on:
      - mysql-server-1
      - mysql-server-2
      - mysql-server-3
```
контейнер выполняет в mysqlsh скрипт setupCluster.js  
подключаясь к первому серверу создает кластер и добавляет в него инстансы 2 и 3  
```js
var dbPass = "OtusLinux2020"
var clusterName = "VogelFrei"

try {
  print('Setting up InnoDB cluster...\n');
  shell.connect('root@mysql-server-1:3306', dbPass)
  var cluster = dba.createCluster(clusterName);
  print('Adding instances to the cluster.');
  cluster.addInstance({user: "root", host: "mysql-server-2", password: dbPass})
  print('.');
  cluster.addInstance({user: "root", host: "mysql-server-3", password: dbPass})
  print('.\nInstances successfully added to the cluster.');
  print('\nInnoDB cluster deployed successfully.\n');
} catch(e) {
  print('\nThe InnoDB cluster could not be created.\n\nError: ' + e.message + '\n');
}
```

## Проверка
после авторизации в ВМ mysql выполняю "mysqlsh" и подключаюсь к localhost:6446  
MySQL root пароль OtusLinux2020  

```bash
[root@mysql ~]# mysqlsh
MySQL Shell 8.0.20

Copyright (c) 2016, 2020, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.
```
```js
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
