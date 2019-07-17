---
title: '''MySQL的基本操作'
date: 2019-07-16 22:55:09
tags:
    - SQL
category: 数据库
---

数据库(database)：保存有组织的数据的容器，数据库有表(table)组成，每个表又由一个或者多个列(column)组成。MySQL 数据库是一种关系型数据库。关系描述的是两个实体之间的对应规则，包括(一对一、一对多、多对多)。在关系型数据库中一行就是一个对象。
<!-- more -->

##关系型数据库的设计都遵守‘三范式’
   * 第一范式： 列不可拆分
   * 第二范式： 唯一标识
   * 第三范式： 引用主键
  后一个范式都是在前一个范式的基础上建立的。

##使用 MySQL
   * 登录 mysql
   '''
   mysql -u 用户名 -p 密码
   '''

   * 退出 mysql
   '''
   exit
   '''

   * 查看所有数据库
   '''
   SHOW DATABASE;
   '''

   * 删除数据库
   '''
   DROP DATABASE 数据库用户名;
   '''

   * 查看当前正在使用的数据库
   '''
   SELECT DATABASE();
   '''

   * 切换数据库
   '''
    USE 数据库名;
   '''

   * 查看某个数据库中的表
   '''
   SHOW TABLES;
   '''

   * 查看表的结构、显示列
   '''
   DESC 表名;
   SHOW columns FROM 表名;
   '''

   * 查看创建数据库、表的 mysql 语句
   '''
   SHOW CREATE DATABASE 库名;
   SHOW CREATE TABLE 表名;
   '''

   * 创建数据库、表
   '''
   CREATE DATABASE 库名;
   CREATE TABLE 表名;
   '''

##数据库的增删改查

  ###改 ALTER
