
# REPMGR+PGPOOL实现双机流式复制、集群


## REPMGR
是一个基于流复制的工具，用于管理和监控PostgreSQL数据库的复制。它可以自动创建和管理主备服务器，并提供了故障检测、自动故障转移、故障恢复等功能。REPMGR可以帮助构建高可用性系统，确保数据库的连续性。

## PGPOOL
是一个连接池和负载均衡器，用于管理和优化与PostgreSQL数据库的连接。它允许多个客户端应用程序共享和复用数据库连接，从而减轻数据库服务器的负载，提高系统的性能和可伸缩性。PGPOOL还可以根据负载情况自动分配查询请求到不同的数据库节点，实现负载均衡。
REPMGR和PGPOOL可以结合使用，构建高可用性的PostgreSQL数据库集群，并提供负载均衡机制，以提高性能和可扩展性



# Start up

git fetch --all && git reset --hard origin/master &&  git pull

> 配置.env环境变量

> 数据卷权限设置

```
sudo chown -R 1001:1001 /home/test/postgresql

```

> 启动

```bash
docker-compose  up  --build  -d
```


> 查看容器

```bash
docker ps --filter name=pg
```

![](https://file0.52tesla.com/xtbf/11b585ec6c294a339872b96320271cd5/1eb0359e)


成功启动后， 可以用127.0.0.1:5999的地址访问数据库，并且自动读写分离, pg-0主库可读可写，pg-1从库只读，两库之间为流式复制，若主库宕机，从库自动升级成主库

```
psql -h 127.0.0.1 -p 5999 -U postgres
```

```
psql -h 127.0.0.1 -p 5999 -U customuser customdatabase

```

![](https://file0.52tesla.com/xtbf/7b1817d71f824bbdbeb0c23fc9e403d6/acfd5aa9)
![](https://file0.52tesla.com/xtbf/f946fe91ae01431c8c22f72741c96b8b/1c012cce)

# Introduction

## PostgreSQL双节点高可用架构


![](https://file0.52tesla.com/xtbf/d666c28d38424c62a13088f2c1c01cf1/c02aaed6)



docker restart pgpool
docker restart pg-1
docker restart pg-0

## 初始化

展示的时候，要经常使用 show pool_nodes， 看每个节点的状态和延时。 如果某个节点一直是down状态或者replication_delay不是0, 停止然后重启


```
docker stop pg-1
docker start pg-1
```

```

docker-compose  down

docker-compose  up  --build  -d

docker ps --filter name=pg

alias pgpool='psql -h 127.0.0.1 -p 5999 -U customuser customdatabase'
alias pg-0='psql -h 127.0.0.1 -p 6000 -U customuser customdatabase'
alias pg-1='psql -h 127.0.0.1 -p 6001 -U customuser customdatabase'

pgpool

show pool_nodes

```




## 故障转移  (先stop pg-1, 恢复pg-1, 再stop pg-0)


https://dbod-user-guide.web.cern.ch/getting_started/HA/pgpool/

pgpool会自动检测到主节点的故障，并在一定时间间隔内（默认为5秒）选择一个可用的备用节点作为新的主节点。
通常情况下，这个过程只需要几秒钟。同时需要考虑到数据同步的问题, 如果在主节点故障之前已经有数据写入到备用节点，
那么在切换主节点时，确保备用节点上的数据是最新的可能需要一些额外的时间。


```


create table game_room
(
    id                bigserial
        primary key,
    room_id           integer
        unique,
    sb                bigint,
    bb                bigint,
    min_buy_in        bigint,
    max_buy_in        bigint,
    timeout           integer,
    max               integer,
    level             smallint,
    room_type         smallint,
    sng_detail        json,
    tournament_buy_in json,
    keep_table_count  bigint,
    room_bg           text,
    table_bg          text,
    is_available      boolean
);

INSERT INTO game_room (id, room_id, sb, bb, min_buy_in, max_buy_in, timeout, max, level, room_type, sng_detail,
                              tournament_buy_in, is_available)
VALUES (1, 1, 1000, 2000, 50000, 300000, 10, 6, 1, 1, null, null, true);


SELECT * FROM game_room;

docker stop pg-1


```


## 负载均衡
```
  psql -h 127.0.0.1 -p 5999 -U postgres -c "select inet_server_addr()"
```

## 流赋值

## 读写分离


https://docs.sqlalchemy.org/en/14/core/engines.html#use-the-connect-args-dictionary-parameter

