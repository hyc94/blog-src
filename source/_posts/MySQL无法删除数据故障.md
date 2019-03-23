---
title: MySQL无法删除数据故障
date: 2019-03-16 16:50:24
tags: mysql
categories: 运维
---

今天一个客户的MySQL生产库出现了业务数据无法删除的故障，记录下这次故障。

* 现场排查
* 事后分析

------

<!-- more -->

## 现场排查

今天一个开发同事向我寻求帮助，说一个客户的MySQL生产库出现了某张表无法删除的故障。具体现象如下：Navicat删除该表时处于卡住的状态，别的测试表删除并没有问题。

排查过程如下：

1. 由于只有单表处于无法删除状态，所以该表可能有被事务占用的情况

2. 查看innodb引擎状态：`show engine innodb status`，由于信息较多，看到了Dead Lock（死锁），忽略了Transactions的具体信息

3. 查看`information_schema.innodb_trx`事务信息，发现了异常记录，事务的开始时间为0000-00-00，虽然trx_tables_in_use和trx_tables_locked为0，但是已经足够引起怀疑了。令人困扰的是thread_id为0。

   {% asset_img WechatIMG79.png %}

4. 接着查看`show full processlist`的结果，也没有发现异常sql

4. 然后再次仔细查看innodb引擎状态：

   {% asset_img WechatIMG80.png %}

   发现Transactions中有刚刚的trx_id为309470323的异常事务，记录的query id为1080

5. 根据query id，使用kill 1080。表删除恢复正常

## 事后分析

1. 由于是单表数据无法删除，应该是单表存在行锁
2. 需要根据innodb引擎的状态，以及innodb_trx和innodb_locks表查看事务信息
3. 然后根据查询的事务进行处理，修改阻塞sql或者kill

  上述情况中出现的innodb_trx表中thread_id为0的异常事务，其产生原因还需要进一步的研究，在网上暂未找到结果。由于该数据库的日志不完善，并且当时存在数据库重启的操作，所以只能暂时记录下。

## 参考链接

https://dbaplus.cn/news-11-974-1.html