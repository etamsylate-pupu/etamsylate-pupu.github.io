---
title: 慢SQL优化 
tags: [sql]
date: 2024-08-03 18:49:32
categories: technique
urlname: 45
---


## MYSQL执行过程及执行顺序




## MYSQL查询语句执行顺序

所有的查询语句都是从FROM开始执行，在执行过程中，每个步骤都会生成一个虚拟表，这个虚拟表将作为下一个执行步骤的输入，最后一个步骤产生的虚拟表即为输出结果。
```
(9) SELECT 
(10) DISTINCT <column>,
(6) AGG_FUNC <column> or <expression>, ...
(1) FROM <left_table> 
    (3) <join_type>JOIN<right_table>
    (2) ON<join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(7) WITH {CUBE|ROLLUP}
(8) HAVING <having_condtion>
(11) ORDER BY <order_by_list>
(12) LIMIT <limit_number>;
```

各关键字说明如下：
- FROM：从指定表选择数据
- ON：指定连接条件
- JOIN：指定要连接的表，通过 FROM 和 JOIN ON 选择需要执行的数据库表 T 和 S，产生笛卡尔积，生成 T 和 S 合并的临时中间表 Temp1，通过 ON 产生临时中间表 Temp2
- 
