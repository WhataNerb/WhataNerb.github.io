## 主从复制，读写分离

对于数据库服务器的构建，为提高数据的安全性和服务的高效性，有一个很常用的概念叫做**“主从复制，读写分离”**

- **主从复制：**将数据库中的数据保存至多个服务器中，并配置服务器之间的主从（master-slave）关系，所有的从机（slave）与主机（master）数据保持一致，主机监听从机的动态
- **读写分离：**将读操作与写操作分离开，使用专门的服务器响应一种操作。通常以主服务器（master）响应写操作，以从服务器（slave）响应读操作

## Redis的实现

Redis的复制系统遵循以下三个主要的机制：

- 在主机与从机连接状态良好时，主机会持续向从机发送指令流来确保同步主机上数据的变化
- 当主从连接中断时，从机会自动重新建立连接，并发起一个局部的再同步：即同步中断期间丢失的指令流
- 当局部再同步失败时，从机会发起一个全局再同步，这时需要主机创建一个全部数据集的快照并发送给从机，然后再保持同步指令流

Redis的复制系统有以下关键的实现细节：

- Redis默认采用**异步复制**，即从机会定期地向主机确认它们接收的数据量
- 一个主机可以拥有多个从机
- 从机之间可以建立连接。除了将多个从机连接到同一主机之外，从机还可以以类似级联的结构连接到其他从机。从Redis 4.0开始，所有子从机将从主机接收完全相同的复制流。
- Redis复制在主机端是非阻塞的。这意味着当一个或多个从机执行初始同步或部分重新同步时，主机将继续处理查询。
- Redis复制在从机端也是非阻塞的。
- 复制可以用于可伸缩性，以便为只读查询提供多个从站（例如，可以将慢速O（N）操作卸载到从站），或者仅用于提高数据安全性和高可用性。
- 可以使用复制来避免让主数据库将完整数据集写入磁盘的成本。但是，必须小心处理此设置，因为重新启动的主服务器将以空数据集开始：如果从服务器尝试与其同步，则从服务器也将被清空。

## 配置方法

1. 打开redis.conf配置文件，找到===NETWORK===配置bind属性，绑定服务器的IP地址

```
################################## NETWORK #####################################

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 loopback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1 (在这里添加此服务器IP地址)
```

2. 开启redis-server

```
redis-server (配置文件路径)
```

3. 开启redis-cli控制台

```
redis-cli -h (服务器IP地址) -p (端口号，默认6379)
```

4. 在从机控制台中输入从机配置命令

```
SLAVEOF (主机IP地址) (主机服务端口)
```

5. 查看当前服务器的主从状态

```
INFO replication
```

## 几种特殊状况

在实际的生产环境中会有各种可能发生的状况，以下举例说明：

1. **主机宕机，从机存活：**从机的地位还是从机（slave）不会改变，原地待命，但是连接状态会由up变为down。若此时使用**SLAVEOF no one**命令可以使从机断开与主机的连接状态，自身晋升为主机
2. **主机宕机后复活：**从机会自动重新和主机建立连接，恢复之前的连接状态
3. **主机存活，从机宕机：**从机会脱离连接结构，其他从机与主机保持正常连接
4. **从机宕机后复活：**默认情况下，从机不会自动和之前的主机建立连接，若要恢复连接，需要手动重连

## 主从复制在主机关闭持久化时的安全性

启用主从复制时，建议开启主从机的持久化功能。如果实在无法做到的话，也应当将服务器配置为关闭自动重启。

为什么关闭持久化在发生重启时很危险呢？请看以下案例：

1. 我们设置A为主机，并且关闭持久化。设置B和C为从机，并从主机A处复制数据
2. A发生错误而崩溃，但是它有自动重启，于是进行重启。然而由于关闭了持久化，重启后数据为空
3. 从机B,C等待到主机A恢复后，自动与主机A重新连接并同步数据，于是数据全部清空。

## 哨兵机制

Redis提供了哨兵（Sentinel）机制，用于监控主从状态并执行容错恢复。其配置方法如下：

1. 创建sentinel.conf文件

2. 配置哨兵，填写内容

   ```
   sentinel monitor 被监控数据库的名字（自己起名） IP地址 端口号 投票数
   ```

   投票数表示，在主机宕机后，从机之间进行投票，超过指定投票数的从机晋升为主机

3. 启动哨兵

   ```
   redis-sentinel sentinel.conf路径
   ```

   