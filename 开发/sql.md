sql语句中各关键字执行顺序如下：
1. form：确定数据来源（表 or 子查询）
2. where：过滤掉不符合条件的数据行
3. group by：数据分组
	1. 分组数据查询时，除了<font color="#ff0000">分组字段之外的字段</font>均需使用聚合函数包裹
4. having：过滤<font color="#ff0000">分组后</font>的数据
5. select：
6. order by：
7. limit
