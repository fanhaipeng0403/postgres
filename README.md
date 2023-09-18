# repmgr+pgpool实现双机流式复制、集群


# Start up

- 配置host文件，执行如下命令

```bash
sudo sh -c 'if ! sudo grep -qxF "$(cat ./host)" /etc/hosts; then sudo cat ./host >> /etc/hosts; fi;'
```

- 配置.env配置环境变量

- 启动master节点
```bash
docker-compose -f docker-compose.master.yml up  --build  -d
```

- 启动slave节点
```bash
docker-compose -f docker-compose.slave.yml up  --build  -d
```

成功启动后， 无论server-0还是server-1中的服务，都可以用127.0.0.1:9999的地址访问数据库，并且自动读写分离。

主库可读可写，从库只读，两库之间为流式复制，若主库宕机，从库自动升级成主库

# Introduction

## PostgreSQL双节点高可用架构


![enter image description here](https://file0.52tesla.com/xtbf/d666c28d38424c62a13088f2c1c01cf1/c02aaed6)



## repmgr
是一个基于流复制的工具，用于管理和监控PostgreSQL数据库的复制。它可以自动创建和管理主备服务器，并提供了故障检测、自动故障转移、故障恢复等功能。REPMGR可以帮助构建高可用性系统，确保数据库的连续性。

## pgpool
是一个连接池和负载均衡器，用于管理和优化与PostgreSQL数据库的连接。它允许多个客户端应用程序共享和复用数据库连接，从而减轻数据库服务器的负载，提高系统的性能和可伸缩性。PGPOOL还可以根据负载情况自动分配查询请求到不同的数据库节点，实现负载均衡。
REPMGR和PGPOOL可以结合使用，构建高可用性的PostgreSQL数据库集群，并提供负载均衡机制，以提高性能和可扩展性