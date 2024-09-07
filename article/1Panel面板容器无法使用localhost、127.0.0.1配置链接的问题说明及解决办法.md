# 报错现象：
安装图床应用时无法连接数据库,但是MySQL进程无报错
mysql：报错信息一览

> SQLSTATE[HY000] [2002] Connection refused


当使用PHP 运行环境/容器安装的应用时,即从1panel应用商店安装(自行安装的应用使用localhost连接不受影响)
MySQL报错

> SQLSTATE[HY000] [2002] Connection refused (SQL: SHOW FULL TABLES WHERE table_type = 'BASE TABLE')

1. 使用localhost 报错无法链接redis、mysql。
2. 使用127.0.0.1报错无法链接redis、mysql。

# 主要原因：

1panel的运行环境为docker创建的容器，故此我们无法通过localhost、127.0.0.1进行直接访问和使用mysql、redis、memache等服务。

这是由于docker每个容器都是单独的一个ip造成的，故此每个容器环境中的127.0.0.1和localhost都是独立的，就相当于两个世界那样，完全无法进行联通。

# 解决办法：

如果使用公网ip、内网ip端是可以进行访问的。但是由于公网访问会造成延迟、内网ip会在容器重启后进行变化，故此不建议通过这两种方式进行解决。

使用容器名称配置连接信息，有效解决办法。
如容器名称如下类型：

复制容器名称并在你的网站配置文件host处填写替换localhost、127.0.0.1：

[![1723907441483.png](http://23.224.239.232:40027/i/2024/08/17/66c0bd740391b.png)](http://23.224.239.232:40027/i/2024/08/17/66c0bd740391b.png)

配置容器名称后再次连接数据库，你会发现后续重启容器和服务器都不需要重新填写配置信息。redis和其他数据库或服务也是如此配置，使用容器名称进行连接。