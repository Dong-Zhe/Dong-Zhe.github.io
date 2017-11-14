---
layout: post
title: How I Rest From Work
date: 2017-09-12 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
tags: [Holidays, Hawaii]
---
1 MySql与传统数据的区别在于存储引擎架构。MySql是三层架构，最上层是连接层，包括安全认证、授权等功能；中间是服务层，大多数功能在这一层实现，包括存储过程、触发器、视图，这一层包含了其核心服务（查询解析、优化、缓存、函数）；底层是存储引擎，负责数据的存储和提取，存储引擎有通用的API（开始事务、提取记录），但不会相互通信，只响应上层请求。这种设计让数据查询/系统任务与数据存储/提取分离，可以根据实际需求选择存储方式。  

2 MySql通过解析树解析查询，并优化重写；优化过程需要存储引擎返回相关开销，所以优化结果受存储引擎影响。SELECT语句会先查询缓存。

3 服务层和存储引擎层都提供并发控制。行锁只由存储引擎提供，支持最大的并发访问，也由最大的锁开销。此外，MySql还提供共享锁、排他锁、表锁。锁的数据量越小，并发度越高，开销越大。

4 事务是一组查询：要么全部成功，要么全部失败。START TRANSACTION;...;COMMIT;，具有原子性（全成功or全回滚）、一致性（COMMIT才产生影响）、隔离性（提交后才对其他事务可见）、持久性（提交后永久保存），简称ACID。事务带来更大的开销。

5 隔离级别：READ UNCOMMITTED，事务的修改中间态对其他事务可见，会产生脏读（针对某条记录）。READ COMMITTED，只能读事务COMMIT以后的记录，不能避免不可重复读（在其他事务提交前读了一次，提交后读了一次，结果不一致，单条记录）。REPEATABLE READ（可重复读），默认，普通索引和无索引情况下使用间隙锁，不能避免幻读（某个范围记录多次读取不一样，由其他事务INSERT和DELETE造成）；SERIALIZEABLE，行锁，可以用MVCC替代。

6 死锁：检测依赖、超时机制；可以对由最少行排他锁的事务回滚。除了数据冲突，还和存储引擎相关。

7 事务日志：内存中保留一份日志，在硬盘上做一个备份；所有行为先在内存的事务日志中记录，COMMIT后再修改相应磁盘数据。

8 混合使用存储引擎，在非事务型的表上的变更无法回滚。InnoDB使用两段锁协议，在加锁阶段可以随时锁定，解锁阶段才能释放，并发好，但有死锁风险。

9 MVCC，两个隐藏列，分别保存创建时间和删除时间。SELECT:查找当前时间大于其创建时间且小于其删除时间的记录；UPDATE：插入记录及记录创建时间；DELETE:更新删除时间；UPDATE：插入新记录，当前时间为创建时间，讲旧记录的删除时间更新为当前时间。

10 转换存储引擎，a.ALTER TABLE X ENGINE = Y b.mysqldump c.CREATE TABLE X LIKE Y。
