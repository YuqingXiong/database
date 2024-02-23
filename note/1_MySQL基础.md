# 单一表格检索

## SELECT

语法：

```mysql
SELECT columns_list
FROM table_name;
```

查询多个字段：

```mysql
SELECT columns_list1, columns_list12
FROM table_name;
```

查询所有字段：

```mysql
SELECT *
FROM table_name;
```

-----

不检索数据，可以省略 FROM 子句：

查时间：

```mysql
SELECT NOW();
```

做数值计算：

```mysql
SELECT 1+2;
```

## WHERE

WHERE 子句可以过滤数据

语法：

```mysql
SELECT columns_list
FROM table_name
WHERE query_condition;
```

例子：

```mysql
SELECT name
FROM student
WHERE age = 18;
```

`AND` ，`OR` 可以组合多个条件进行查询

```mysql
SELECT name
FROM student
WHERE age = 18 AND score = 100;
-- -----------------
SELECT name
FROM student
WHERE age = 18 OR score = 100;
```

**比较运算符**

上面的例子只用了等于号 `=`

还有：

| 比较运算符    | 说明                               | 举例                        |
| ------------- | ---------------------------------- | --------------------------- |
| `=`           | 等于                               | `age = 18`                  |
| `<>`          | 不等于                             | `age <> 18`                 |
| `!=`          | 不等于                             | `age != 18`                 |
| `>`           | 大于，通常用于比较数字或者日期     | `age > 18`                  |
| `>=`          | 大于等于，通常用于比较数字或者日期 | `age >= 18`                 |
| `<`           | 小于，通常用于比较数字或者日期     | `age < 18`                  |
| `<=`          | 小于等于，通常用于比较数字或者日期 | `age <= 18`                 |
| **`IN`**      | **判断值是否在一个集合中**         | **`age IN (18, 19)`**       |
| `NOT IN`      | 判断值是否不在一个集合中           | `age NOT IN (18, 19)`       |
| **`BETWEEN`** | **判断值是否介于两个数中间**       | **`age BETWEEN 16 AND 18`** |
| **`LIKE`**    | **模糊匹配**                       | **`name LIKE 'A%'`**        |

## LIKE

需要一个字符串模式，进行匹配：

```mysql
column_name LIKE 'pattern'
```

- `%` 表示匹配任意多个字符
- `_` 表示匹配任意单个字符
- `\` 用于转义
  - `\_` 表示匹配 `_` 
  - `\%` 表示匹配 `%` 
- 使用通配符匹配文本时，不区分大小写

## EXISTS

EXISTS 需要一个子查询作为参数：

```mysql
SELECT column_name
FROM table_name
WHERE EXISTS(subquery)
```

例子：

```mysql
SELECT *
FROM student
WHERE EXISTS(
	SELECT *
    FROM course
    WHERE course.student_id = student.id
)
```

## ORDER BY

用于指定排序的字段，和排序方式

```mysql
SELECT column_name1, column_name2,...
FROM my_table_name
[WHERE query_clause] 
ORDER BY
	column_name1 [ASC|DESC],
	column_name2 [ASC|DESC],
	...;
```

- `ASC`：升序排序
- `DESC`：降序排序

### FIELD 自定义排序

```mysql
SELECT *
FROM course
ORDER BY FIELD(course_name, "语文", "数学", "英语");
```

## LIMIT 

限制返回的行数，也可以规定起始偏移数：

```mysql
SELECT column1...
FROM my_table
LIMIT [row_offset] row_count;
```

- `row_offset` 指定要返回的第一行的偏移量，是可选的，默认为 0
- `row_count` 指定返回的最大行数

常用于分页：

```mysql
SELECT *
FROM my_table
LIMIT (page_num-1)*show_num show_num;
```

## DISTINCT

`DISTINCT` 关键字用于去除重复行结果，放在 `SELECT` 后面

```mysql
SELECT DISTINCT name
FROM student
```

# 多表格检索

## JOIN

`JOIN` 分内连接，外连接（左连接，右连接），交叉连接

- 交叉连接是指求两个表的笛卡尔积，返回两个表的行和列所有的组合

```mysql
SELECT *
FROM 
table1 CROSS JOIN table2
```

- 内连接是指求两个表的交集，可以指定连接条件：

```mysql
SELECT student.name, student_course.score
FROM student
INNER JOIN student_course ON student.sid = student_course.sid
-- INNER 可以省略
```

- 左连接，是以左边表为基础匹配右边表的每一行，不管条件是否被满足左边表的数据都会被返回：

```mysql
SELECT *
FROM my_table1
LEFT JOIN my_table2 ON my_table1.id1 = my_table2.id2
```

- 右连接，是以右边表为基础匹配右边表的每一行，不管条件是否被满足右边表的数据都会被返回：

```mysql
SELECT *
FROM my_table1
RIGHT JOIN my_table2 ON my_table1.id1 = my_table2.id2
```

---

`JOIN` 还可以实现自连接：

例如一个员工表里包括所有员工，现查询员工的信息和上级信息：

```mysql
SELECT *
FROM employees e
JOIN employees m
ON e.reports_to = m.employee_id -- 上级的id等于员工id
```

----

`JOIN` 还可以级联，同时连接 3 张，或者 n 张表格：

```mysql
SELECT *
FROM my_table1
JOIN my_table2 ON my_table1.id1 = my_table2.id2
JOIN my_table3 ON my_table1.id1 = my_table3.id3
...;
```

---

连接条件可以复合多个条件，每个条件之间用 `AND` / `OR` /... 连接：

常用于复合主键表格之间的连接

```mysql
SELECT *
FROM table1
JOIN table2 ON table1.id = table2.id AND table1.xxx = table2.xxx
```

---

`WHERE` 来写条件其实是隐式连接

## UNION

用于合并两个 `SELECT` 语句操作的结果（合并行）

```mysql
SELECT * FROM table1 WHERE xxx
UNION
SELECT * FROM table2 WHERE xxx
```

注意：

- `UNION` 中的 `SELECT` 语句的**列数和列顺序必须相同**
- `UNION` 默认是 `UNION DISTINCT` 结果去重的，不去重的话用 `UNION ALL`

# 增删改（插入 更新 删除）



## 列属性

- INT(11)：整数值

- VARCHAR(50)：可变字符，最大存储50个字符，如果只有5个字符，则只存储这5个，不会浪费空间

- CHAR(50)：如果只有5个字符，则会用空格符填满剩余的45个字符

## INSERT

```mysql
INSERT INTO table_name [column_1, column_2, ...]
VALUES (value_1, value_2, ...);
```

## UPDATE

```mysql
UPDATE table_name
SET 
	column_name1 = value1,
	column_name2 = value2,
	...
[WHERE xxx] -- 指定更新行
```

返回修改的行数

例子：（子查询更新）

```mysql
UPDATE customer
SET store_id = (
    SELECT store_id
    FROM store
    ORDER BY RAND()
    LIMIT 1
  )
WHERE store_id IS NULL;
```

## DELETE

```mysql
DELETE FROM my_table_name
[WHERE clause] -- 指定删除的行
[ORDER BY ..] -- 指定删除行的顺序
[LIMIT row_count] -- 指定删除的最大行数
```

返回删除的行数

# 聚合函数

- `SUM()`: 求总和
- `AVG()`: 求平均值
- `MAX()`: 求最大值
- `MAX()`: 求最小值
- `COUNT()`: 计数

```mysql
SELECT 
	SUM(xxxx),
	AVG(xxx),
	MAX(xx) [AS max_number],
	...
FROM my_table
[WHERE xxxx]
```

## GROUP BY

`GROUP BY` 可以根据指定的字段或者表达式对查询的结果进行分组

例子：查询总消费金额排名前十的客户

```mysql
SELECT customer_id, SUM(amount) as total
FROM payment
GROUP BY customer_id
ORDER BY total DESC
LIMIT 10
```

## HAVING

`HAVING` 子句用于过滤分组后的结果，`WHERE` 过滤分组前的数据

`HAVING` 逻辑表达式中的字段只能用分组使用的字段和聚合函数

例如：获取位于VA州的且消费总额大于100的顾客信息

```mysql
SELECT
	C.customer_id,
	C.first_name,
	C.last_name,
	SUM(oi.quantity * oi.unit_price) AS total_sales 
FROM customers c
JOIN orders o USING(customer_id)
JOIN order_items oi USING(order_id)
WHERE state='VA'
GROUP BY 
	C.customer_id,
	C.first_name,
	C.last_name
HAVING total_sales > 100
```

# 复杂查询

## 子查询

查找所有比生菜贵的产品（id=3)：

```mysql
SELECT *
FROM products
WHERE unit_price > (
	SELECT unit_price
    FROM products
    WHERE product_id = 3
)
```

所有收入在平均线以上的雇员：

```mysql
SELECT *
FROM employees
WHERE salary > (
	SELECT AVG(salary)
    FROM employees
)
```

## IN 运算符

````mysql
column_name NOT IN/ IN (xxxx)
````

IN 的 子查询返回了一个列表（集合）的值

查找所有没有被买过的产品:

```mysql
SELECT * 
FROM products
WHERE product_id NOT IN (
    SELECT DISTINCT product_id
	FROM order_items
    )
```

查找没有发票的客户：

```mysql
SELECT *
FROM clients
WHERE client_id NOT IN (
	SELECT DISTINCT client_id
    FROM invoices
)
```

## 子查询与JOIN连接

可以用连接来重写上面的例子，例如：

```mysql
SELECT *
FROM clients
LEFT JOIN invoices USING(client_id)
WHERE invoice_id is NULL
```

具体应该选择哪种方式需要根据可读性和执行效率进行选择

查找所有订购了生菜（id=3）的顾客：

```mysql
SELECT *
FROM customers
WHERE customer_id IN (
	SELECT o.customer_id
    FROM order_items oi
    JOIN orders o USING (order_id)
    WHERE product_id = 3
)
```

```mysql
SELECT DISTINCT customer_id, first_name, last_name
FROM customers c
JOIN orders o USING (customer_id)
JOIN order_items oi USING (order_id)
WHERE oi.product_id = 3
```

## ALL 关键字

查询大于客户3所有发票金额的发票：

```mysql
SELECT *
FROM invoices
WHERE invoice_total > (
	SELECT MAX(invoice_total)
    FROM invoices
    WHERE client_id = 3 
)
```

可以使用 ALL 关键字实现：

```mysql
SELECT *
FROM invoices
WHERE invoice_total > ALL(
	SELECT invoice_total
    FROM invoices
    WHERE client_id = 3 
)
```

```mysql
invoice_total > ALL(159, 130, 167) -- invoice_total与括号内的每个数字进行比较
```

## ANY 关键字

查询至少有两个发票的客户：

```mysql
SELECT *
FROM clients
WHERE client_id IN (
	SELECT clinet_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
)
```

ANY 写法：

```mysql
SELECT *
FROM clients
WHERE client_id = ANY(
	SELECT clinet_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
)
```

## 相关子查询

选择工资超过部门平均工资的员工：

```mysql
SELECT * 
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE office_id = e.office_id
)
```

> 子查询引用了主查询出现的别名，**子查询会在著查询每一行的层面执行**（所以相关子查询会很慢）

获取高于客户平均发票金额的发票：

```mysql
SELECT *
FROM invoices i
WHERE invoice_total > (
	SELECT AVG(invoice_total)
    FROM invoices
    WHERE client_id = i.client_id
)
```

## EXISTS 运算符

获取有发票的客户：

IN 关键字/JOIN实现：

```mysql
SELECT * 
FROM clients
WHERE client_id IN (
	SELECT DISTINCT client_id
    FROM invoices
)
```

EXISTS 实现：查看是否有符合条件的记录行

```mysql
SELECT *
FROM clients c
WHERE EXISTS(
	SELECT client_id
    FROM invoices
    WHERE client_id = c.client_id
)
```

> `id IN(1,2,,3,4,...)` 如果 IN 括号里的数据非常大，那么就会生成一个非常大的列表，影响了查询的性能
>
> `EXIST(xx)` 只需要找到一条记录就行

找到没有被订购的产品：

```mysql
SELECT *
FROM products p
WHERE NOT EXISTS(
	SELECT product_id
    FROM order_items
    WHERE product_id = p.product_id
)
```

## SELECT 中的子查询

```mysql
SELECT 
	invoice_id,
	invoice_total,
	(SELECT AVG(invoice_total) FROM invoices) AS invoice_average,
	invoice_total-(SELECT invoice_average) AS difference,
FROM invoices
```

> - 这里不能直接写 `AVG(invoice_total)` 这样只会返回一个值，而写子查询语句返回的是一列值
> - `SELECT invoice_average` 是上一个子查询的简写

查询客户的发票id，发票总额，发票平均值，以及总额与平均值的差值：

```mysql
SELECT 
	client_id, 
	name, 
	(SELECT SUM(invoice_total) 
     	FROM invoices
    	WHERE client_id = c.client_id) AS total_sales
    (SELECT AVG(invoice_total) FROM invoices) AS average
    (SELECT total_sales - average)	AS difference
FROM clients c
```

---

TOP - K 问题：

求每个科目前 3 名的学生：

```mysql
SELECT r.cname, s.name, r.score
FROM student s
JOIN result r USING(sid)
WHERE (
	SELECT COUNT(*)
    FROM result
    WHERE courseid = r.courseid AND score > r.score
) < 3
ORDER BY r.courseid, r.score DESC
```

无法使用 LIMIT 3 ，这样会截取掉相同分数的学生

## FROM中的子查询

```mysql
SELECT *
FROM (
    SELECT 
        client_id, 
        name, 
        (SELECT SUM(invoice_total) 
            FROM invoices
            WHERE client_id = c.client_id) AS total_sales
        (SELECT AVG(invoice_total) FROM invoices) AS average
        (SELECT total_sales - average)	AS difference
    FROM clients c
) AS sales_summary
WHERE total_sales is NOT NULL
```

# 基本函数

## 数值函数

四舍五入 `ROUND()`：

```mysql
SELECT ROUND(5.73) -- 6
SELECT ROUND(5,73, 1) -- 5.7
SELECT ROUND(5,73, 2) -- 5.73
```

大于等于这个数的最小整数 `CEILING()` ：

```mysql
SELECT CELING(5.2) -- 6
```

小于等于这个数的最大整数 `FLOOR()`：

```mysql
SELECT FLOOR(5.2) -- 5
```

取绝对值 `ABS()`：

```mysql
SELECT ABS(-5.2) -- 5
```

生成 0-1 的随机浮点数 `RAND()`：

```mysql
SELECT RAND()
```

## 字符串函数

获取字符串长度 `LENGTH()`：

```mysql
SELECT LENGTH('sky') -- 3
```

大小写转换：

```mysql
SELECT UPPER('sky') -- SKY
SELECT LOWER('Sky') -- sky
```

...

## 日期函数

```mysql
SELECT NOW(), CURDATE(), CURTIME()
-- 2024-02-20 14:40:05, 2024-02-20, 14:40:05

SELECT YEAR(NOW()), MONTH(NOW()), ...
-- 2024, 2
```

- 格式化日期
- 计算日期

# 视图

[MySQL 视图 (sjkjc.com)](https://www.sjkjc.com/mysql/view/)

MySQL 是一种常用的关系型数据库管理系统，提供了 `CREATE VIEW` 语法，用于创建视图（View）。视图是一种虚拟的表，实际上并不存储数据，而是从一个或多个表中派生出来的查询结果集，具有与表相似的结构。通过创建视图，可以将复杂的查询操作封装成一个简单的视图，方便用户进行查询和数据访问。

创建视图：

```mysql
CREATE VIEW view_name AS
SELECT xxxx
```

删除视图：

```mysql
DROP VIEW view_name
```

更新视图：

```mysql
-- 创建可更新视图：
CREATE VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions
WITH [CASCADED | LOCAL] CHECK OPTION; -- 防止行消失
```

`CASCADED` 表示更新限制会传递到视图引用的所有表，而 `LOCAL` 表示更新限制仅应用于当前视图

# 存储

## 过程

创建存储过程：

```mysql
DELIMITER $$ -- 定义新的分隔符
CREATE PROCEDURE get_clients() -- 创建新的存储过程
BEGIN 
	SELECT * FROM clients;
END$$

DELIMITER;
```

使用存储过程：

```mysql
CALL get_clients()
```

删除存储过程：

```mysql
DROP PROCEDURE IF EXISTS get_clients
```

## 参数

传递参数：

```mysql
DELIMITER $$ 
CREATE PROCEDURE get_clients_by_state
(
    state CHAR(2)	-- 参数（2个字符的字符串）
)
BEGIN 
	SELECT * FROM clients c
	WHERE c.state = state; -- 使用参数
END$$

DELIMITER;
```

参数验证：

```mysql
CREATE PROCEDURE make_payment
(
    invoice_id INT,
    payment_amouont DECIMAL(9,2),
    payment_date DATE
)
BEGIN
	IF payment_amount <= 0 THEN
		SIGNAL SQLSTATE '22003' 
			SET MESSAGE_TEXT = 'Invalid payment amount';
	END IF;
	
	UPDATE invoices i
	SET 
		i.payment_amouont = payment_amouont,
		i.payment_date = payment_date
	WHERE i.invoice_id = invoicec_id;
END
```

输出参数：

```mysql
CREATE PROCEDURE get_unpaid_invoices_for_client
(
    -- 定义参数
	client_id INT, 
    invoices_count TINYINT,
    invoices_total DECIMAL(9,2)
)
BEGIN
    SELECT COUNT(*), SUM(invoice_total)
    INTO invoices_count, invoices_total -- 定义哪些参数是输出的
    FROM invoices i
    WHERE i.client_id = client_id 
    	AND payment_total = 0;
END

```

![image-20240222145929660](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402221459848.png)

## 变量

```mysql
-- User or session variables
SET @invoices_count = 0

-- Local variable，只在存储过程中才能使用，存储过程结束就无法使用了
CREATE PROCEDURE get_risk_factor()
BEGIN
	-- 声明本地变量 
	DECLARE risk_factor DECIMAL(9,2);
	DECLARE invoices_total DECIMAL(9,2);
	DECLARE invoices_count INT:
	
	SELECT count(*), SUM(invoice_total)
	INTO invoices_count, invoices_total
	FROM invoices;
	-- 使用本地变量
	SET risk_factor = invoices_total / invoices_count * 5;
	
	SELECT risk_factor;
END
```

## 函数

```mysql
CREATE FUNCTION get_risk_factor_for_client
(
    client_id INT
)
RETURNS INTEGER
-- DETERMINISTIC
READS SQL DATA -- 需要读 SQL 数据
BEGIN
	DECLARE risk_factor DECIMAL(9,2);
	DECLARE invoices_total DECIMAL(9,2);
	DECLARE invoices_count INT:
	
	SELECT count(*), SUM(invoice_total)
	INTO invoices_count, invoices_total
	FROM invoices i
	WHERE i.client_id = client_id;

	SET risk_factor = invoices_total / invoices_count * 5;
	RETURN IFNULL(risk_factor, 0);
END
```

使用函数：

```mysql
SELECT
	client_id,
	name,
	get_risk_factor_for_client(client_id)
FROM clients

-- 删除函数
DROP FUNCTION function_name
```

# 触发器

触发器是在插入，更新和删除语句前后自动执行的一堆SQL代码

通常用触发器实现数据一致性

```mysql
CREATE TRIGGER xxxx
xxx
...
```

# 事件

根据技术执行的任务或者一堆SQL代码

```mysql
DELIMITER $$
-- 每年删除过期审计行
CREATE EVENT yearly_delete_stale_audit_rows
ON SCHEDULE
	-- AT '2019-05-01'
	EVERY 1 YEAR STARTS '2019-01-01' ENDS '2029-01-01'
DO BEGIN
	DELETE FROM payment_audit
	WHERE action_date < NOW() - INTERVAl 1 YEAR
END $$
DELIMITER;
```

查看，删除，更改事件

```mysql
SHOW EVENTS;
SHOW EVENTS LIKE 'yearly%';

DROP EVENT IF EXISTS yearly_delete_stale_audit_rows

ALTER EVENT xxxx ENABLE;
```

# 事务

希望所有的更改作为一个单元一起成功或失败

创建事务：

```mysql
START TRANSACTION -- 事务格式

INSERT INTO orders (customer_id, order_date, status)
VALUES (1, '2024-02-22', 1)

INSERT INTO order_items
VALUES (LAST_INSERT_ID(), 1, 1, 1);

-- 事务格式
-- COMMIT; -- 直接提交
ROLLBACK; -- 事务回滚，退回所有事务
```

## 并发和锁定

```mysql
STARTE TRANSACTION
UPDATE customers
SET points = points + 10
WHERE customer_id = 1;
COMMIT;
```

当一个事务要修改一个行时，就会给这个行上锁，避免其他事务对它进行更改

---

并发问题：

- 当没上锁时，后面的事务覆盖了前面事务的结果，丢失了更i性能的数据：

![image-20240222160908222](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402221609408.png)

- 脏读：更新前被另一个事务读了：

![image-20240222160949184](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402221609311.png)

避免上述问题，建立事务隔离

## 隔离

事务的隔离级别：

- 读已提交：只能读已经提交了的数据
- 可重复读：保证读到的同一数据一致，避免其他事务对数据的修改影响到自己的事务
- 序列化：防止幻读
  - 可以知晓其他事务对自身事务用到数据的改动
  - 如果有其他事务改动了数据，就会等待其他事务完成，再完成自己的事务
  - 这种按照顺序的方式进行等待执行称为事务的序列化隔离

![image-20240222161811287](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402221618407.png)

查看事务的隔离级别：

```mysql
SHOW VARIABLES LIKE 'transation_isolation';
```

设置下一个事务的隔离级别：

```mysql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 为会话和未来所有事务设置
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 为所有会话和未来所有事务设置
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

## 死锁

```mysql
USE sql_store;
-- TRANSACTION 1
START TRANSACTION;
-- 事务 1 锁住了 customers 中的记录
UPDATE customers SET state = 'VA' WHERE customer_id = 1; 
-- 被事务 2 锁住了，无法更改
UPDATE orders SET status = 1 WHERE order_id = 1;
COMMIT;
```

```mysql
USE sql_store;
-- TRANSACTION 2
START TRANSACTION;
-- 事务 2 锁住了 orders 中的记录
UPDATE orders SET status = 1 WHERE order_id = 1;
-- 被事务 1 锁住了，无法更改
UPDATE customers SET state = 'VA' WHERE customer_id = 1;
COMMIT;
```

事务 1，2互相死锁

# 数据类型

- `CHAR(x)` 不可变长度字符串
- `VARCHAR(x)` 可变长度字符串 max:65535 characters(~64KB)
- `MEDIUMTEXT` max:16MB
- `LONGTEXT` max:4GB
- `TINYTEXT` max:255 bytes
- `TEXT`  max: 64KB 

---

- `TINYINT` 1b [-128,127]
- `unsigned tinyint` [0,255]
- `INT(x)`  ...

![image-20240223102544380](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231025742.png)

---

- `DECIMAL(p,s)` p:总位数，s:精度；
- `DEC`
- `NUMERIC`
- `FIXED`
- `FLOAT ` 4b
- `DOUBLE `8b

---

- `BOOL`
- `BOOLEAN` 

---

枚举/集合

`ENUM('small', 'medium', 'large')`

不推荐使用，可以另建一张表，存储每个枚举对象对应的索引

---

日期和时间

- `DATE` 存储日期，不包含时间
- `TIME` 存储时间
- `DATETIME` 8b
- `TIMESTAMP` 时间戳，4b 只能存储到2038年
- `YEAR` 

---

BLOBS 二进制长对象，存储图像，pdf，视频

- `TINYBLOB` 255b
- `BLOB` 65kb
- `MEDIUMBLOB` 16mb
- `LONGBLOB` 4GB

尽力不要把文件存储在数据库中，应该存储在文件系统中

---

JSON类型

![image-20240223104019447](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231040966.png)

# 设计数据库

## 主键外键

[MySQL PRIMARY KEY 主键 (sjkjc.com)](https://www.sjkjc.com/mysql/primary-key/)

[MySQL FOREIGN KEY 外键 (sjkjc.com)](https://www.sjkjc.com/mysql/foreign-key/)

## 三大范式

- **第一范式：要求一行中每个单元格都应该有单一值，且不能出现重复列**

下面是不允许的：

| tag  | tag  | tag  |
| ---- | ---- | ---- |

| id   | xx   | tags        |
| ---- | ---- | ----------- |
|      |      | (1,2,3,...) |

![image-20240223110215802](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231102297.png)

courses 表中的 tags列违反了第一范式，这一列会存储多个 tag 值

添加链接表 course_tags，删除 tags列，用链接表表示 course 和 tag的多对多关系：

![image-20240223110146210](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231101680.png)

- **第二范式：每张表都是用来单独描述某个实体的**

instructor 列违反了第二范式，instructor 是一个老师的名字，他是老师的属性，而不是课程的数量，而且名字更新很麻烦。

应该单独建一个讲师表格，存储每个讲师的名字等属性，再在courses表中关联 讲师instructor 表中的 id

![image-20240223111831920](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231118366.png)

- 第三范式：表中的列不应派生其他列

# 创建表

[MySQL CREATE TABLE 创建表 (sjkjc.com)](https://www.sjkjc.com/mysql/table-create/)

[MySQL DROP TABLE 删除表 (sjkjc.com)](https://www.sjkjc.com/mysql/table-drop/)

[MySQL ALTER TABLE 修改表 (sjkjc.com)](https://www.sjkjc.com/mysql/table-alter/)

# 索引

当根据某个查询条件查询数据时很慢，可以在查询相关的列上创建索引。相当于存储了一个字典的目录在内存中，可以根据排序好的索引找到整行数据。

![image-20240223143631836](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231436215.png)

![image-20240223143707189](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231437342.png)

索引是一种类似二叉树的数据结构，例如 B-Tree，提高了从表检索数据行的速度，但也增加了额外的写入和存储

---

## 创建索引

```mysql
-- 为 customers 中的 state 列创建一个索引 idx_state
CREATE INDEX idx_state ON customers (state);
```

```mysql
-- 找到积分大于 1000 的顾客
CREATE INDEX idx_points ON customers (points);
SELECT customer_id FROM coustomers WHERE points > 1000;
```

## 查看索引

```mysql
SHOW INDEXES IN customers;

ANALYZE TABLE customers;

```

## 前缀索引

对于 char varchar blob 把字符串整个当作索引会占很多空间，可以截取部分进行索引的建立

```mysql
CREATE INDEX idx_lastname ON customers (last_name(20));
```

找到合适的截取长度?

找到截取不同前缀长度的字符串能对应的唯一值

```mysql
SELECT 
	COUNT(LEFT(last_name, 1)),
	COUNT(LEFT(last_name, 5)),
	COUNT(LEFT(last_name, 10))
FROM customers
```

![image-20240223145246292](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231452436.png)

截取5个字符只比截取10个字符的唯一值少30，但是节省了一半的存储空间，所以截取5个更划算

## 全文索引

搜索包含 `react redux` 关键字的博客，react 和 redux 出现的顺序不限制，其中一个出现就行，中间可以有其他单词....

 ```mysql
 CREATE FULLTEXT INDEX idx_title_body ON posts(title, body);
 
 SELECT *
 FROM posts
 WHERE MATCH(title, body) AGAINST('react redux');
 ```

`MATCH(title, body) AGAINST('react redux')` 可以计算相关性得分

`MATCH(title, body) AGAINST('react -redux +form')` 不包含 redux ，一定包含 form

## 复合索引

```mysql
-- 同时根据两个条件进行查询，mysql只会选择其中一个索引，另一个查询条件依然会扫描剩下的所有行
SELECT customer_id FROM customers
WHERE state = 'CA' AND points > 1000;

-- 建立复合索引解决上面的问题：
CREATE INDEX idx_state_points ON customers (state, points);
```

复合索引中的列顺序：

- 将最常用的列放在前面
- 基数更高的列放在前面（唯一值多的列放在前面）

```mysql
-- 查询位于 CA 且以 A开头的顾客
SELECT customer_id
FROM customers
WHERE state = 'CA' AND last_name LIKE 'A%';
```

为上面的查询建立索引时，如果基于上面的准则会发现，last_name的基础比state多，但是如果把 last_name 放在复合索引前面的话，需要扫描所有以A开头的顾客

而如果 state 放在复合索引前面的话，由于每个state内字符串是按照字典序排列好的，所以查找A开头的顾客只用扫描连续的几行

```mysql
CREATE INDEX idx_state_lastname ON customers(state, last_name);
```

![image-20240223151723025](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231517227.png)

## 索引无效

```mysql
-- 注意这里用了 OR，虽然建立了索引，但是依然扫描了全表
SELECT customer_id FROM customers
WHERE state = 'CA' OR points > 1000;

-- 改为两个利用索引的小查询
SELECT customer_id FROM customers
WHERE state = 'CA'
UNION
SELECT customer_id FROM customers
WHERE  points > 1000;
```



```mysql
SELECT customer_id FROM customers
WHERE points + 10 > 2010; -- 索引无法作用，扫描了全表

SELECT customer_id FROM customers
WHERE points > 2000; -- 提出单独的列，可以用索引
```

## 索引排序

```mysql
-- 存在一个 (a,b)的索引
-- 排序时也需要按照索引建立的顺序进行排序才更快 order (a,b)
```

## 覆盖索引

包含所有满足查询需要的数据的索引称为覆盖索引

这样可以在不读表的情况下查询数据

```mysql
SELECT customer_id, state -- id和state都是索引
FROM customers
ORDER BY state;
```

## 维护索引

可能创建了重复索引，多余索引

![image-20240223154010252](https://xiongyuqing-img.oss-cn-qingdao.aliyuncs.com/img/202402231540437.png)
