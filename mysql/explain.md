## explain后会有 

好像别人写得更好： [MySQL explain详解](https://www.cnblogs.com/butterfly100/archive/2018/01/15/8287569.html)

+----+-------------+------------------------+------------+------+---------------+------+---------+------+--------+----------+-------+
| id | select_type | table                  | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra |
+----+-------------+------------------------+------------+------+---------------+------+---------+------+--------+----------+-------+

#### 下面我们逐已解释上面每个字段的意义：

**id**,表示