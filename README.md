# MYSQL窗口函数

|     函数名    	| 功能|
| ------------- | -------------------------		|
| CUME_DIST()   | 累计分布值。                   	|
| DENSE_RANK()  | 当前行位于分区的排名，无断档。	|
| RANK()        | 当前行位于分区的排名，有断档。	|
| PERCENT_RANK()| 排名所占百分比。                |
| FIRST_VALUE() | 窗口中第一行内容。              	|
| LAST_VALUE()  | 窗口中最后一行内容。            	|
| LAG()         | 当前行前的N行内容。             	|
| LEAD()        | 当前行后的N行内容。             	|
| NTH_VALUE()   | 窗口中第N行的内容              	|
| NTILE()       | 返回当前行在分区中的桶号        	|
| ROW_NUMBER()  | 行号                          	|

## 1. CUME_DIST
CUME_DIST: 累计分布值
返回当前分区中小于等于当前值的百分比，由于是百分比，取值范围0-1
举例
表名: numbers

| row_number | val |
| ---------- | --- |
|       1    |  1  |
|       2    |  1  |
|       3    |  2  |
|       4    |  3  |
|       5    |  3  |
|       6    |  3  |
|       7    |  4  |
|       8    |  4  |
|       9    |  5  |

建表SQL:
```SQL
DROP TABLE IF EXISTS  `numbers`;
CREATE TABLE `numbers` (`row_number` INT NULL, `val` INT NULL);
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('1', '1');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('2', '1');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('3', '2');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('4', '3');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('5', '3');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('6', '3');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('7', '4');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('8', '4');
INSERT INTO `numbers` (`row_number`, `val`) VALUES ('9', '5');
```

执行下面的查询语句: 
```SQL
SELECT val,
	ROW_NUMBER()   OVER w AS 'row_number',
	CUME_DIST()    OVER w AS 'cume_dist',
	PERCENT_RANK() OVER w AS 'percent_rank'
FROM `numbers`
	WINDOW w AS (ORDER BY val);
```

结果为:

| val | row_number |      cume_dist    | percent_rank |
| --- | ---------- | ----------------- | ------------ |
|  1  |     1      | 0.2222222222222222| 0            |
|  1  |     2      | 0.2222222222222222| 0            |
|  2  |     3      | 0.3333333333333333| 0.25         |
|  3  |     4      | 0.6666666666666666| 0.375        |
|  3  |     5      | 0.6666666666666666| 0.375        |
|  3  |     6      | 0.6666666666666666| 0.375        |
|  4  |     7      | 0.8888888888888888| 0.75         |
|  4  |     8      | 0.8888888888888888| 0.75         |
|  5  |     9      | 1                 | 1            |

数据分析: 
2个1，1个2，3个3，2个4，1个5
||
| ------------------------------------------------- |
|小于等于1的，占比2/9=0.2222222222222222				|
|小于等于1的，占比2/9=0.2222222222222222				|
|小于等于2的，占比(2+1=3)/9=0.3333333333333333		|
|小于等于3的，占比(2+1+3=6)/9=0.6666666666666666		|
|小于等于3的，占比(2+1+3=6)/9=0.6666666666666666		|
|小于等于3的，占比(2+1+3=6)/9=0.6666666666666666		|
|小于等于4的，占比(2+1+3+2=8)/9=0.8888888888888888	|
|小于等于4的，占比(2+1+3+2=8)/9=0.8888888888888888	|
|小于等于5的，占比(2+1+3+2+1=9)/9=1					|

## 2. DENSE_RANK vs RANK
DENSE_RANK: 在当前分区中的排名，排名无断档

RANK: 在当前分区中的排名，有断档。

所谓断档即出现并列的排名的情况，是否会出现某个排名缺失，比如1中提出的表格，执行下面的SQL语句
```SQL
SELECT
	val,
	ROW_NUMBER() OVER w AS 'row_number',
	RANK()       OVER w AS 'rank',
	DENSE_RANK() OVER w AS 'dense_rank'
FROM `numbers`
	WINDOW w AS (ORDER BY val);
```
则会出现以下结果
| val  | row_number | rank | dense_rank |
| ---- | ---------- | ---- | ---------- |
|    1 |          1 |    1 |          1 |
|    1 |          2 |    1 |          1 |
|    2 |          3 |    3 |          2 |
|    3 |          4 |    4 |          3 |
|    3 |          5 |    4 |          3 |
|    3 |          6 |    4 |          3 |
|    4 |          7 |    7 |          4 |
|    4 |          8 |    7 |          4 |
|    5 |          9 |    9 |          5 |

数据分析: 
根据表中可以看到会有并列第一的情况（2个并列第一），当出现并列第一的时候，rank的下一个排名则从3开始，而dense_rank则从2开始。

## 3. PERCENT_RANK
返回该排名所位于的百分比，公式为：(rank - 1) / (rows - 1)，数据可参考1.中提到的PERCENT_RANK()

## 4. FIRST_VALUE vs LAST_VALUE
FIRST_VALUE 获取当前窗口的第一行的值

LAST_VALUE 获取当前窗口的最后一行的值

表名: number_list

| index |
| ----- |
|   1   |
|   2   |
|   3   |
|   4   |
|   5   |
|   6   |
|   7   |
|   8   |
|   9   |

建表SQL:
```SQL
DROP TABLE IF EXISTS  `number_list`;
CREATE TABLE `number_list` (`index` INT NOT NULL);
INSERT INTO `number_list` (`index`) VALUES ('1');
INSERT INTO `number_list` (`index`) VALUES ('2');
INSERT INTO `number_list` (`index`) VALUES ('3');
INSERT INTO `number_list` (`index`) VALUES ('4');
INSERT INTO `number_list` (`index`) VALUES ('5');
INSERT INTO `number_list` (`index`) VALUES ('6');
INSERT INTO `number_list` (`index`) VALUES ('7');
INSERT INTO `number_list` (`index`) VALUES ('8');
INSERT INTO `number_list` (`index`) VALUES ('9');
```

执行SQL:
```SQL
SELECT
	FIRST_VALUE(`index`) OVER w AS `FIRST_VALUE`,
	LAST_VALUE(`index`) OVER w AS `LAST_VALUE`
FROM `number_list`
	WINDOW w AS (ORDER BY `index`);
```

结果为:
| FIRST_VALUE | LAST_VALUE |
| ----------- | ---------- |
|      1      |      1     |
|      1      |      2     |
|      1      |      3     |
|      1      |      4     |
|      1      |      5     |
|      1      |      6     |
|      1      |      7     |
|      1      |      8     |
|      1      |      9     |


数据分析:
FIRST_VALUE始终为1，原因是当前的查询中，第一条记录永远都是1，但是LAST_VALUE并不为9，原因是窗口数据在不断更新，由于按``index``列进行查询，所以最后一个会是1-9，若执行
```SQL
SELECT
	FIRST_VALUE(`index`) OVER w AS `FIRST_VALUE`,
	LAST_VALUE(`index`) OVER w AS `LAST_VALUE`
FROM `number_list`
	WINDOW w AS (ORDER BY `index` DESC);
```
则 FIRST_VALUE永远为9， LAST_VALUE出现为9-1。所以一定要注意LAST_VALUE为当前的最后值，而非执行后的最后值。若我们改变表顺序，则会出现更有趣的现象。

建表SQL:
```SQL
DROP TABLE IF EXISTS `number_list2`;
CREATE TABLE `number_list2` (`id` INT NOT NULL AUTO_INCREMENT, `index` INT NOT NULL, PRIMARY KEY (`id`));
INSERT INTO `number_list2` (`index`) VALUES ('5');
INSERT INTO `number_list2` (`index`) VALUES ('2');
INSERT INTO `number_list2` (`index`) VALUES ('3');
INSERT INTO `number_list2` (`index`) VALUES ('7');
INSERT INTO `number_list2` (`index`) VALUES ('6');
INSERT INTO `number_list2` (`index`) VALUES ('9');
INSERT INTO `number_list2` (`index`) VALUES ('8');
INSERT INTO `number_list2` (`index`) VALUES ('4');
INSERT INTO `number_list2` (`index`) VALUES ('1');
```

表名: number_list2

|  id   | index |
| ----- | ----- |
|   1   |   5   |
|   2   |   2   |
|   3   |   3   |
|   4   |   7   |
|   5   |   6   |
|   6   |   9   |
|   7   |   8   |
|   8   |   4   |
|   9   |   1   |

执行SQL:
```SQL
SELECT
	FIRST_VALUE(`index`) OVER w AS `FIRST_VALUE`,
	LAST_VALUE(`index`) OVER w AS `LAST_VALUE`
FROM `number_list2`
	WINDOW w AS (ORDER BY `id`);
```
结果为：
| FIRST_VALUE | LAST_VALUE |
| ----------- | ---------- |
|      5      |      5     |
|      5      |      2     |
|      5      |      3     |
|      5      |      7     |
|      5      |      6     |
|      5      |      9     |
|      5      |      8     |
|      5      |      4     |
|      5      |      1     |

数据分析:
由于按照``id``排序，所以第一个永远是5，最后按照id的顺序打印出来。

若想返回最终窗口内的值，则需要``ORDER BY NULL``

SQL语句为:
```SQL
SELECT
	FIRST_VALUE(`index`) OVER w AS `FIRST_VALUE`,
	LAST_VALUE(`index`) OVER w AS `LAST_VALUE`
FROM `number_list2`
	WINDOW w AS (ORDER BY NULL);
```

结果为
| FIRST_VALUE | LAST_VALUE |
| ----------- | ---------- |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |
|      5      |      1     |

## 5. LAG vs LEAD
``LAG``为返回当前行的前一行的值，``LEAD``为返回当前行的后一行的值。

``LAG``与``LEAD``可以有3个入参。

第一个参数为列名。

第二个参数为可选参数，既第N行，若想返回当前行前的两行则可写为``LAG(`column`, 2)``和``LEAD(`column`, 2)``，该参数可以为0，0表示当前行，但是不能是负数。

第三个参数为可选参数，当结果为空时候的默认值。

建表SQL:
```SQL
DROP TABLE IF EXISTS `number_list2`;
CREATE TABLE `number_list2` (`id` INT NOT NULL AUTO_INCREMENT, `index` INT NOT NULL, PRIMARY KEY (`id`));
INSERT INTO `number_list2` (`index`) VALUES ('5');
INSERT INTO `number_list2` (`index`) VALUES ('2');
INSERT INTO `number_list2` (`index`) VALUES ('3');
INSERT INTO `number_list2` (`index`) VALUES ('7');
INSERT INTO `number_list2` (`index`) VALUES ('6');
INSERT INTO `number_list2` (`index`) VALUES ('9');
INSERT INTO `number_list2` (`index`) VALUES ('8');
INSERT INTO `number_list2` (`index`) VALUES ('4');
INSERT INTO `number_list2` (`index`) VALUES ('1');
```

表名: number_list2

|  id   | index |
| ----- | ----- |
|   1   |   5   |
|   2   |   2   |
|   3   |   3   |
|   4   |   7   |
|   5   |   6   |
|   6   |   9   |
|   7   |   8   |
|   8   |   4   |
|   9   |   1   |

SQL语句为:
```SQL
SELECT
	LAG(`index`) OVER w AS `LAG`,
	LEAD(`index`) OVER w AS `LEAD`
FROM `number_list2`
	WINDOW w AS (ORDER BY `id`);
```

执行结果为:
| LAG  | LEAD |
| ---- | ---- |
| NULL |   2  |
|  5   |   3  |
|  2   |   7  |
|  3   |   6  |
|  7   |   9  |
|  6   |   8  |
|  9   |   4  |
|  8   |   1  |
|  4   | NULL |


数据分析:
执行结果显而易见，既按照``id``的顺序进行排序，然后输出结果，``LAG``为当前行的前一行，``LEAD``为当前行的后一行，没有时返回默认值``NULL``。

3个参数均填入值举例
```SQL
SELECT
	LAG(`index`, 2, "没有值") OVER w AS `LAG`,
	LEAD(`index`, 2, "没有值") OVER w AS `LEAD`
FROM `number_list2`
	WINDOW w AS (ORDER BY `id`);
```

执行结果为:
|  LAG  |  LEAD |
| -----	| ----- |
| 没有值	|   3   |
| 没有值	|   7   |
|   5   |   6   |
|   2   |   9   |
|   3   |   8   |
|   7   |   4   |
|   6   |   1   |
|   9   | 没有值	|
|   8   | 没有值	|

数据分析:
数据顺序为 5 2 3 7 6 9 8 4 1，第一行中，5往前减少2行没有值，所以LAG为``没有值``，往后2行的值为3。剩下以此类推。

## 6. NTH_VALUE
该函数为返回第N行中的值。

建表SQL:
```SQL
DROP TABLE IF EXISTS `number_list2`;
CREATE TABLE `number_list2` (`id` INT NOT NULL AUTO_INCREMENT, `index` INT NOT NULL, PRIMARY KEY (`id`));
INSERT INTO `number_list2` (`index`) VALUES ('5');
INSERT INTO `number_list2` (`index`) VALUES ('2');
INSERT INTO `number_list2` (`index`) VALUES ('3');
INSERT INTO `number_list2` (`index`) VALUES ('7');
INSERT INTO `number_list2` (`index`) VALUES ('6');
INSERT INTO `number_list2` (`index`) VALUES ('9');
INSERT INTO `number_list2` (`index`) VALUES ('8');
INSERT INTO `number_list2` (`index`) VALUES ('4');
INSERT INTO `number_list2` (`index`) VALUES ('1');
```

表名: number_list2

|  id   | index |
| ----- | ----- |
|   1   |   5   |
|   2   |   2   |
|   3   |   3   |
|   4   |   7   |
|   5   |   6   |
|   6   |   9   |
|   7   |   8   |
|   8   |   4   |
|   9   |   1   |

SQL语句为:
```SQL
SELECT
	NTH_VALUE(`index`, 5) OVER w AS `id=5`
FROM `number_list2`
	WINDOW w AS (ORDER BY `id`);
```

执行结果为:
|  id=5 |
| -----	|
|  NULL |
|  NULL |
|  NULL |
|  NULL |
|   6   |
|   6   |
|   6   |
|   6   |
|   6   |

数据分析:
数据顺序为``5 2 3 7 6 9 8 4 1``，由于排序的前4行还不知道第五行的值，所以返回``NULL``，当第5行的值出现后，恒定显示第5行的值。如果``ORDER BY NULL``，则运算完所有的值，获取第5行的值，返回都为6，在实际应用中，若想去除重复，则可是用
```SQL
SELECT
	DISTINCT NTH_VALUE(`index`, 5) OVER w AS `id=5`
FROM `number_list2`
	WINDOW w AS (ORDER BY NULL);
```

### 6. NTILE
将数据平均分配到N个桶中并返回桶编号。


建表SQL:
```SQL
DROP TABLE IF EXISTS `number_list2`;
CREATE TABLE `number_list2` (`id` INT NOT NULL AUTO_INCREMENT, `index` INT NOT NULL, PRIMARY KEY (`id`));
INSERT INTO `number_list2` (`index`) VALUES ('5');
INSERT INTO `number_list2` (`index`) VALUES ('2');
INSERT INTO `number_list2` (`index`) VALUES ('3');
INSERT INTO `number_list2` (`index`) VALUES ('7');
INSERT INTO `number_list2` (`index`) VALUES ('6');
INSERT INTO `number_list2` (`index`) VALUES ('9');
INSERT INTO `number_list2` (`index`) VALUES ('8');
INSERT INTO `number_list2` (`index`) VALUES ('4');
INSERT INTO `number_list2` (`index`) VALUES ('1');
```

表名: number_list2

|  id   | index |
| ----- | ----- |
|   1   |   5   |
|   2   |   2   |
|   3   |   3   |
|   4   |   7   |
|   5   |   6   |
|   6   |   9   |
|   7   |   8   |
|   8   |   4   |
|   9   |   1   |

SQL语句为:
```SQL
SELECT
	`index`,
	NTILE(2) OVER w AS 'ntile2',
	NTILE(4) OVER w AS 'ntile4'
FROM `number_list2`
	WINDOW w AS (ORDER BY NULL);
```

执行结果为:
| index | ntile2 | ntile4 |
| ----- | ------ | ------ |
|   5   |    1   |    1   |
|   2   |    1   |    1   |
|   3   |    1   |    1   |
|   7   |    1   |    2   |
|   6   |    1   |    2   |
|   9   |    2   |    3   |
|   8   |    2   |    3   |
|   4   |    2   |    4   |
|   1   |    2   |    4   |


数据分析:
数据顺序为``5 2 3 7 6 9 8 4 1``，NTILE(2)表示只分2个桶，所以``5 2 3 7 6``被分到#1号桶中（若分不开则前面的桶会多分到），``9 8 4 1``被分到#2号桶。NTILE(4)表示分为4个桶，#1号桶有``5 2 3``，#2号桶有``7 6``，#3号桶有``9 8``，#4号桶有``4 1``。

## 7. ROW_NUMBER
获取当前窗口中的行号。

表名: number_list

| index |
| ----- |
|   1   |
|   2   |
|   3   |
|   4   |
|   5   |
|   6   |
|   7   |
|   8   |
|   9   |

建表SQL:
```SQL
DROP TABLE IF EXISTS  `number_list`;
CREATE TABLE `number_list` (`index` INT NOT NULL);
INSERT INTO `number_list` (`index`) VALUES ('5');
INSERT INTO `number_list` (`index`) VALUES ('2');
INSERT INTO `number_list` (`index`) VALUES ('3');
INSERT INTO `number_list` (`index`) VALUES ('7');
INSERT INTO `number_list` (`index`) VALUES ('6');
INSERT INTO `number_list` (`index`) VALUES ('9');
INSERT INTO `number_list` (`index`) VALUES ('8');
INSERT INTO `number_list` (`index`) VALUES ('4');
INSERT INTO `number_list` (`index`) VALUES ('1');
```

SQL语句为:
```SQL
SELECT
	ROW_NUMBER() OVER w AS 'row_number',
	`index`
FROM `number_list`
	WINDOW w AS (ORDER BY NULL);
```

执行结果为:

| row_number | index |
| ---------- | ----- |
|     1      |   5   |
|     2      |   2   |
|     3      |   3   |
|     4      |   7   |
|     5      |   6   |
|     6      |   9   |
|     7      |   8   |
|     8      |   4   |
|     9      |   1   |


数据分析:
row_number为每行的行号，感觉没啥好讲的:)
