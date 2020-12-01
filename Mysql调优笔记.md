---
title: Mysql调优笔记
date: 2020-11-23 15:36:39
tags:
- linux
- 学习
- 数据库
- Mysql
categories : "数据库"
---

> - Mysql调优学习笔记
> - 记录数据库的相关知识
> - 具体问题具体分析

<!-- more-->

# 性能监控
## show profile（慢慢被废弃，mysql5可以踏实用）
- 命令：set profiling=1; 打开
- 命令：show profiles;   显示语句运行时间
- 命令：show profile (for query $id);    显示所有操作时间（开始，结束，网络发送数据）, 不加括号的话，默认查询最近的一条

还有其他参数可以看官方文档。

## performance schema(性能模式)
mysql的performance shcema用于监控mysql server，在一个较低级别的运行过程中的资源消耗、资源等待等情况。

- performane_schema这个库，使用的是performance_schema这个存储引擎

如果需要监控数据库性能使用情况，可以查看这个库下的所有表，具体表的内容可以Google搜。

## show processlist
查看当前mysql数据服务有多少连接

优化的时候，用的比较少，一般实际场景会使用数据库连接池。

# schema与数据类型优化
## 数据类型优化
- 更小的通常更好，但是需要确保没有低估数据范围。
- 使用自建的date类型存储时间，而不是string类型
- 尽量避免NULL，查询中包含NULL，对mysql来说很难优化
- 如果确定数据类型不变的话，可以在mysql中使用enum类型

## 合理使用范式和反范式
在企业中，需要根据具体场景，来使用范式或者反范式。

## 主键选择
1. 代理主键：与业务无关，id
2. 自然主键：事物属性相关联，如uid
3. 推荐使用代理主键，不与业务耦合，并且通用，减少编码数量。

# 执行计划
## id
id: 当前计划的执行顺序，如果都一样的话，从上往下执行。id值越大，对应优先级越高，优先执行

## select_type
分辨查询的类型，普通查询还是联合查询还是子查询

## type（重要）
访问方式，以何种方式去访问数据，如全表扫描（ALL）

- all 全表扫描
- index 全索引扫描，效率比all好一些。
- range 表示利用索引查询时，限制了范围（**最低要满足的标准**）
- index_subquery 利用索引关联子查询，不再扫描全表
- unique_subquery，类似index_subquery，但是用的是唯一索引
- index_merge，查询过程中，多个索引组合使用
- const，只命中一个匹配行(**最好的情况**)

## possible_keys
当前查询计划，可能用到的索引

## key
当前查询计划，实际应用到的索引

## key_len
索引中使用的字节数，可以推出使用了哪个索引

## ref
显示索引的哪一列被使用了，如果可能的话，是一个常数

## rows
大致估算出需要读取的行数，不一定准

## extra
- using filesort 表示mysql无法利用索引排序，只能用排序算法。一般和order by相关，出现这个不大好
- using temporary，建立临时表保存中间结果，完成后删除
- using index，覆盖索引
- using where，利用where进行条件过滤
- impossible where，where语句结果总是false

# 索引优化小细节
1. 尽量把计算的数据放在业务层（索引列查询不要使用表达式），而不是在语句体现。（可能会导致执行计划有差别，影响效率）
2. union all , in , or都能够用索引，但是推荐用in
3. 索引默认是升序的，使用order by排序的时候，尽量用上索引排序
4. 索引最多用于一个范围列，即用了> , < 后，后面的判断条件不会走索引
5. join表连接的时候，不要超过三张表
6. 单表索引控制在5个以内
7. 单索引字段不允许超过5个

# 索引监控信息
- show status like 'Handler_read%'

# 记录
- MVCC并不是在所有情况下解决了幻读，而是只在读的情况解决了幻读