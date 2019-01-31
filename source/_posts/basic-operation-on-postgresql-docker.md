---
title: 使用 PostgreSQL Docker 的基本操作
date: 2019-02-01 00:43:02
tags:
- PostgreSQL10
- MySQL
- CentOS7
---

這兩天都在把 MySQL10 的資料搬到 PostgreSQL10，稍微紀錄一下開發筆記，開發環境是 CentOS7 ，已經很習慣用 docker 來做開發了，如果沒有 docker ，裝個 db 卸載個 db 該有多麻煩...

## PostgreSQL Docker

先拉 image，然後用互動模式跑起來測測看
```
$ docker pull postgres:10-alpine
$ docker run -it --rm --name postgresql \
    -e POSTGRES_PASSWORD=${USER_PASS} -e POSTGRES_USER=${USER_NAME} -d \
    -p 5432:5432 \
    --net host \
    -v /etc/hosts:/etc/hosts:ro \
    -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
    -v /usr/share/zoneinfo/Asia/Taipei:/etc/localtime:ro \
    -v ${WORKING_DIR}/data:/var/lib/postgresql/data \
    postgres:10-alpine
```

Docker 如果能順利 run 起來應該沒有太大問題，既然都開進 docker 裡了，而且我們的 data 是 mount 在 host 上的，可以順便修改`/var/lib/postgresql/data/pg_hba.conf`，讓 host 上的 pg client 連 docker 裡的 pg server 可以免密碼，這招在 docker compose 上使用的時候，也可以省很多次打密碼的時間…

然後在開發 vm 上要裝 postgresql 的 client ， CentOS7 預設的版本是 9 ，我們的 docker 是用 10 的，所以 client 也最好用 10

```
$ sudo rpm -Uvh https://yum.postgresql.org/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
$ sudo yum install -y postgresql10
```

然後那一包 bin 都在 `/usr/pgsql-10/bin/` 裡，為了和預設的 pg9 cleint 做區別，我這裡都做 alias
```
$ sudo ln -s /usr/pgsql-10/bin/psql /usr/local/bin/psql10
$ sudo ln -s /usr/pgsql-10/bin/pg_dump /usr/local/bin/pg_dump10
$ sudo ln -s /usr/pgsql-10/bin/pg_restore /usr/local/bin/pg_restore10
```

稍微測一下
```
$ psql10 --version
psql (PostgreSQL) 10.6
```

## Docker compose with MySQL and PostgreSQL

稍微寫一個 docker-compose.yml 同時開這兩個 db 的範例
```
version: '2.1'

services:
    mysql:
        container_name: mysql 
        hostname: mysql.mydomain
        restart: always
        privileged: true
        networks:
            mynet:
                ipv4_address: 172.20.0.2
        ports:
            - 3306:3306
        image: mariadb/server:10.2
        environment:
            - MYSQL_ROOT_PASSWORD=${ROOT_PASS}
            - MYSQL_USER=${USER_NAME}
            - MYSQL_PASSWORD=${USER_PASS}
        volumes:
            - /etc/hosts:/etc/hosts:ro
            - /sys/fs/cgroup:/sys/fs/cgroup:ro
            - /usr/share/zoneinfo/Asia/Taipei:/etc/localtime:ro
            - ${WORKING_DIR}/mysql/data:/var/lib/mysql
            - ${WORKING_DIR}/mysql/log:/var/log/mysql
    postgresql:
        container_name: postgresql
        hostname: postgresql.mydomain
        restart: always
        privileged: true
        networks:
            mynet:
                ipv4_address: 172.20.0.3
        image: postgres:10-alpine
        ports:
            - 5432:5432 
        environment:
            - POSTGRES_DB=${MY_DB}
            - POSTGRES_USER=${USER_NAME}
            - POSTGRES_PASSWORD=${USER_PASS}
        volumes:
            - /etc/hosts:/etc/hosts:ro
            - /sys/fs/cgroup:/sys/fs/cgroup:ro
            - /usr/share/zoneinfo/Asia/Taipei:/etc/localtime:ro
            - ${WORKING_DIR}/postgresql/data/mnt:/var/lib/postgresql/data 
networks:
    mynet:
        name: mynet
        enable_ipv6: false
        ipam:
            config:
            - subnet: 172.20.0.0/16

```

這樣會建立一個 bridge network 是 172.20.0.0/16 ，上面兩個 container 可以和 host 在這個網段下溝通，也可以透過 host 連外，記得裝一下 MySQL 的 client 連連看，這邊就不多寫了，剛剛那個`/var/lib/postgresql/data/pg_hba.conf` ，也可以添加這句，達到免密碼登入 pg

```
host    all             all             172.20.0.1/16           trust
```

## pgloader

由上面的筆記走到這邊，應該 MySQL 和 PostgreSQL 的 container 都建好了，假設現在 MySQL 裡面有一些資料，我們想要直接 copy 到 pg ，最暴力的那種，細節先不多說

以上述例子提到的 ip 和 docker network 來說
```
$ docker run --security-opt seccomp=unconfined \
    --rm --net mynet --name pgloader \
    dimitri/pgloader:latest pgloader \
    mysql://${USER_NAME}:${USER_PASS}@172.19.0.2/${MY_DB} \
    postgresql://${USER_NAME}@172.20.0.3/${MY_DB}
```

如果一切順利的話，這樣就搬完了，當然你不會對自動幫你 copy 的 pg schema 太滿意 XDD

## pg 的 dump 和 restore

剛剛我們有針對 pg10 的 dump 和 restore 做過 alias ，現在可以直接拿來用

```
$ pg_dump10 -h pg_ip -U your_username -p 5432 -F c -v -f "your_backup.bak" pg_db_name 
$ pg_restore10 -v -h pg_ip -U your_username -p 5432 -d pg_db_name "your_backup.bak"
```

其中 dump 的 -F 參數改成 p ，可以輸出 sql 檔

以上，快速做個整理
