---
layout: post
title: MySQL分页+排序数据重复的问题
date: 2019-12-12 11:03:43.000000000 +08:00
tags: MySQL
---

>最近工作中遇到一个使用MySQL分页+排序时，MySQL返回数据重复的问题。虽然以现在的水平还不知道具体原因，但是还是先要记录一下问题以及解决方案。待以后技术水平达到一定水准或者找到原因再回来更新这篇文章。

### 一、问题描述

有一张数据表：业务线数据表 `business_line` 的内容如下：

```bash
mysql> SELECT * FROM business_line;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
|  1 | 业务线1        |         10 |      0 | 2019-12-10 11:07:04 | 2019-12-12 10:47:04 |
|  2 | 业务线2        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:21 |
|  3 | 业务线3        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:22 |
|  4 | 业务线4        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:23 |
|  5 | 业务线5        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:24 |
|  6 | 业务线6        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:25 |
|  7 | 业务线7        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:26 |
|  8 | 业务线8        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:27 |
|  9 | 业务线9        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:28 |
| 10 | 业务线10       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:30 |
| 11 | 业务线11       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:34 |
| 12 | 业务线12       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:37 |
+----+---------------+------------+--------+---------------------+---------------------+
```

>`created_time`字段设计为`DEFAULT CURRENT_TIMESTAMP`，且由于表中的数据是使用`INSERT`语句批量一次性插入数据表中，所以每一行数据中的`created_time`字段值都是`2019-12-10 11:07:04`，这是问题的前提。

我的需求是在前端页面使用分页 + 按`created_time`降序排序的查询规则来查询并展示这个表的数据，分页语句为：`SELECT * FROM business_line ORDER BY created_time DESC LIMIT #{count} OFFSET #{offset};`

现在查询第一页的数据（每页10条，offset为 0）：

```bash
mysql> SELECT * FROM business_line ORDER BY created_time DESC LIMIT 10 OFFSET 0;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
| 12 | 业务线12       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:37 |
| 10 | 业务线10       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:30 |
|  9 | 业务线9        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:28 |
|  8 | 业务线8        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:27 |
|  7 | 业务线7        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:26 |
|  6 | 业务线6        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:25 |
|  5 | 业务线5        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:24 |
|  4 | 业务线4        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:23 |
|  3 | 业务线3        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:22 |
|  2 | 业务线2        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:21 |
+----+---------------+------------+--------+---------------------+---------------------+
```

嗯 ... 看着好像没啥问题。继续查询第二页数据（每页10条，offset为 10）：

````bash
mysql> SELECT * FROM business_line ORDER BY created_time DESC LIMIT 10 OFFSET 10;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
|  2 | 业务线2        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:21 |
| 12 | 业务线12       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:37 |
+----+---------------+------------+--------+---------------------+---------------------+
````

问题来了：第二页居然查出重复的数据，已知`业务线2`和`业务线12`在第一页查询的时候已经查出来了。总共就两页数据，反倒是`业务线1`和`业务线11`始终没有查出来！多么窒息的操作 ...

### 二、问题原因追踪

翻阅MySQL官网使用手册，在 [8.2.1.17 LIMIT Query Optimization](https://dev.mysql.com/doc/refman/5.7/en/limit-optimization.html) 一章中找到一些原因：

>If you combine LIMIT `row_count` with ORDER BY, MySQL stops sorting as soon as it has found the first `row_count` rows of the sorted result, rather than sorting the entire result. If ordering is done by using an index, this is very fast. If a filesort must be done, all rows that match the query without the LIMIT clause are selected, and most or all of them are sorted, before the first `row_count` are found. After the initial rows have been found, MySQL does not sort any remainder of the result set.

看上面这段文字说明：大体意思是，若使用 LIMIT #{count} + ORDER BY 的组合查询，MySQL只要找到`count`条的数据就会立即停止排序，而非将整个结果集全部排序。

>If multiple rows have identical values in the ORDER BY columns, the server is free to return those rows in any order, and may do so differently depending on the overall execution plan. In other words, the sort order of those rows is nondeterministic with respect to the nonordered columns.

这段则说的是，如果 ORDER BY 所排序的字段中存在多行的值相同，针对未指定排序的字段，MySQL返回的结果行数据的顺序是不确定的。

另外一点影响返回数据顺序的因素是 LIMIT 语句，ORDER BY 的时候，后面有 LIMIT 和没有 LIMIT 语句时返回数据的顺序是不一样的。

```bash
ORDER BY 后面没有跟 LIMIT 语句时返回的数据顺序：

mysql> SELECT * FROM business_line ORDER BY created_time;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
|  1 | 业务线1        |         10 |      0 | 2019-12-10 11:07:04 | 2019-12-12 10:47:04 |
| 11 | 业务线11       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:34 |
| 10 | 业务线10       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:30 |
|  9 | 业务线9        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:28 |
|  8 | 业务线8        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:27 |
|  7 | 业务线7        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:26 |
|  6 | 业务线6        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:25 |
|  5 | 业务线5        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:24 |
|  4 | 业务线4        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:23 |
|  3 | 业务线3        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:22 |
|  2 | 业务线2        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:21 |
| 12 | 业务线12       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:37 |
+----+---------------+------------+--------+---------------------+---------------------+

ORDER BY 后面跟 LIMIT 语句时返回的数据顺序：

mysql> SELECT * FROM business_line ORDER BY created_time LIMIT 5;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
| 12 | 业务线12       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:37 |
|  2 | 业务线2        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:21 |
|  3 | 业务线3        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:22 |
|  4 | 业务线4        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:23 |
|  5 | 业务线5        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:24 |
+----+---------------+------------+--------+---------------------+---------------------+
```

### 三、问题解决方案

当指定的排序字段在每一行数据或者再多行数据中的值都一样，MySQL的排序可能就会包含不确定性，就像问题中描述的现象一样：分页返回了重复的数据。为了消除这种不确定性，保证返回的数据顺序是确定的，可以额外使用其他具有唯一性的字段作为排序字段，例如自增ID。

所以根据上述问题中的场景，既然指定的排序字段 `created_time` 已经不可避免地在每行数据中包含了相同的值，那需要再额外使用一个其值具备唯一性的字段自增ID来作为排序字段，得到的分页结果将会是跟预期一样正常了：

现在查询第一页的数据（每页10条，offset为 0）：注意额外加了 `id` 作为排序字段

```bash
mysql> SELECT * FROM business_line ORDER BY created_time DESC, id LIMIT 10 OFFSET 0;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
|  1 | 业务线1        |         10 |      0 | 2019-12-10 11:07:04 | 2019-12-12 10:47:04 |
|  2 | 业务线2        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:21 |
|  3 | 业务线3        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:22 |
|  4 | 业务线4        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:23 |
|  5 | 业务线5        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:24 |
|  6 | 业务线6        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:25 |
|  7 | 业务线7        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:26 |
|  8 | 业务线8        |         11 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:27 |
|  9 | 业务线9        |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:28 |
| 10 | 业务线10       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:30 |
+----+---------------+------------+--------+---------------------+---------------------+
```

继续查询第二页数据（每页10条，offset为 10）：注意额外加了 `id` 作为排序字段

```bash
mysql> SELECT * FROM business_line ORDER BY created_time DESC, id LIMIT 10 OFFSET 10;
+----+---------------+------------+--------+---------------------+---------------------+
| id | buz_line_name | company_id | status | created_time        | updated_time        |
+----+---------------+------------+--------+---------------------+---------------------+
| 11 | 业务线11       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:34 |
| 12 | 业务线12       |          1 |      0 | 2019-12-10 11:07:04 | 2019-12-12 11:41:37 |
+----+---------------+------------+--------+---------------------+---------------------+
```

至此，分页数据重复的问题得到解决。

`ORDER BY created_time DESC, id` 语句中，MySQL会优先使用 `created_time` 字段进行降序排序，当遇到 `created_time` 字段值一样时，会使用 `id` 进行升序排序（不指定则默认`ASC`），从而解决了 ORDER BY + LIMIT 组合使用时分页数据重复的问题了。