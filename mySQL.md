非DBA  和JAVA开发相关的

1、mysql 架构介绍

2、索引优化分析  相关crud 写sql 查sql   索引相关

3、查询截取分析 （查找相关慢的sql,定位sql,将sql重写或者改造）

4、锁机制（行锁 表锁）

5、主从复制



TCL  事务

事务的ACID属性

A Atomicity[ætəˈmɪsəti] 事务不可分割  都发 都不发  原子性

C Consisteny[kənˈsɪstənsi] 事务使数据库从一个一致性状态 切换到另一个一致性状态 一致性

I  Isolcation[ˌaɪsəˈleɪʃn]  事务之间不能干扰  隔离级别  隔离性

D durability[dərəˈbɪlɪti] 事务一旦提交 对数据库的改变就是永久性的   持久性



事务创建

​		隐式事务 ：事务没有明显的开启和结束的标记

​		insert  update delete

​		显示事务：

​		set autocommit =0；开启事务

​		sql语句

​		rollback||commit ; 结束事务



事务隔离级别

​		事务间不采用隔离机制
​				脏读：读取未提交的字段，读到临时且无效的字段   针对更新

​				不可重复度：重复度值不同

​				幻读：重复读取数据 会多读出数据  针对插入

​		

​		设置事务间的隔离级别

​			  查看隔离级别：SELECT @@tx_isolation

​              设置当前连接隔离级别： set transaction isolation  level  read commited;

​			  设置全局隔离级别： set global  transaction isolation  level read  committed;			

​				

















### mysql架构介绍

mysql简介 

mysqlLinux安装

​		rpm -ivh Mysql.rpm

​		自启mysql chkconfig mysql on

​		数据库存在硬盘的 /var/lib/mysql

​		/var/lib/mysql 数据库文件存放路径

​		/usr/share/mysql 配置文件目录

​		/usr/bin 相关命令目录

​		/etc/init.d/mysql 启停相关脚本

Mysql配置文件

​		二进制日志文件:log-bin  用于主从复制

​		错误日志:log-error 默认关闭 记录严重警告和错误信息，每次启动和关闭的详细信息等

​		查询日志:log 默认关闭 记录查询的sql语句 开启会强敌mysql整体性能

​		数据文件

​				frm文件  存放表结构

​				myd文件  存放表数据

​				myi文件   查找表数据的索引

Mysql逻辑架构

​		Connectors连接层

​		Connection Pool 连接池

​		Management Services& Utilities 工具 

​		SQL Interface

​		Parser  转换

​		Optimizer 优化

​		Caches&Buffers 缓存

​		Pluggable Storage Engines 可拔插存储引擎

​		File system 

​		File&Logs

​		插件式的 存储引擎框架将  查询处理   和其他的系统任务以及数据的   存储提取相分离



​		连接层

​				包含客户端和连接服务，包含本地sock通信和大多数基于客户端工具实现的类似于tcp/ip的通信

​				主要完成一些类似于连接处理、授权认证、及相关的安全方案

​				该层引入线程池的概念，为通过认证安全接入的客户端提供线程

​				在该层伤可以实现基于SSL的安全连接，服务器也会为安全接入的每个客户端验证它具有的操作权限

​		服务层

​				完成大多服功能，如SQL接口并完成缓存的查询，SQL的分析和优化及部分内置函数的执行

​				实现跨存储引擎的功能 如过程、函数等等

​				该层内，服务器解析查询并创建相应的内部解析树，并对起完成相应的优化如确定查询表的顺序，是否  		利用索引，最后生成相应的执行操作。如果select会查询内部缓存



​		引擎层

​				存储引擎层，负责Mysql中数据的存储和提取 服务器通过API和存储引擎进行通信

​		存储层

​				数据存储层，将数据存志在运行于裸设备的文件系统之上，并完成于存储引擎的交互

​		

mysql存储引擎

​				InnoDB、MyISAM

| 对比项   | MyISAM                                         | InnoDB                                                       |
| -------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 主外键   | 不支持                                         | 支持                                                         |
| 事物     | 不支持                                         | 支持                                                         |
| 行表锁   | 表锁，一条记录也会锁住整个表，不适合高并发操作 | 行锁，操作时只锁一行，不对其他行有影响，适合高并发操作       |
| 缓存     | 只缓存索引，不缓存真实数据                     | 不仅仅缓存 索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性影响 |
| 表空间   | 小                                             | 大                                                           |
| 关注点   | 性能                                           | 事物                                                         |
| 默认安装 | Y                                              | Y                                                            |



​		



### 索引优化分析

性能下降SQL变慢

常见通用Join查询

索引简介

性能分析

索引优化





### 查询截取分析

查询优化

慢查询日志

批量数据脚本

Show PrOFILE

全局查询日志



### 锁机制

行锁

表锁

页锁





### 主从复制









