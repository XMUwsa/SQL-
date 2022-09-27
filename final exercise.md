## 1.各部门工资最高的员工（难度：中等）
**问题**：创建两个表。Employee表，包含所有员工信息，每个员工有其对应的 Id, salary和 department Id。department表，包含部门号和部门名。然后找出每个部门工资最高的员工。

![image](https://user-images.githubusercontent.com/83053244/192276472-1e49a6b2-eb03-496a-acac-b1a67bdf6fbe.png)


**分析思路**：先找出每个部门最高工资是多少；
然后按照部门id将最高工资作为新的一列添加到employee表，并筛选出salary与max_salary相等的行；
最后将筛选结果与department表联结。本题考查了两点：自联结和多表联结

**代码**：
```sql
#创建两表
CREATE TABLE employee (
	Id INT NOT NULL,
	people VARCHAR ( 100 ) NOT NULL,
	Salary INT,
	DepartmentId INT,
PRIMARY KEY ( Id, people ));
INSERT INTO employee
VALUES
	( 1, 'Joe', 70000, 1 ),
	( 2, 'Henry', 80000, 2 ),
	( 3, 'Sam', 60000, 2 ),
	( 4, 'Max', 90000, 1 );
CREATE TABLE department ( Id INT NOT NULL, site VARCHAR ( 100 ) NOT NULL );
INSERT INTO department
VALUES
	( 1, 'IT' ),
	( 2, 'Sales' );
```
``` sql
#找出各部门工资最高员工
SELECT
	p4.site AS department,
	p3.people AS employee,
	p3.Salary 
FROM
	(
	SELECT
		p1.people,
		p1.DepartmentId,
		p1.Salary,
		p2.max_salary 
	FROM
		employee AS p1
		INNER JOIN ( SELECT DepartmentId, max( Salary ) AS max_salary FROM employee GROUP BY DepartmentId ) AS p2 ON p2.DepartmentId = p1.DepartmentId 
	WHERE
		p2.max_salary = p1.Salary 
	) AS p3
	INNER JOIN department AS p4 ON p3.DepartmentId = p4.Id;
```
**代码注意点**：
``` sql
#按第一行这样写，输出错误的人名对应max_salary。一定要注意有group by时，除了聚合结果，group by子句和select子句的变量要一样。
SELECT people, max( Salary ) AS max_salary FROM employee GROUP BY DepartmentId
SELECT DepartmentId, max( Salary ) AS max_salary FROM employee GROUP BY DepartmentId
```
**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192179907-bf6fa58b-af1d-4839-826c-fa35d5d17e3e.png)

## 2.换座位（难度：中等）
**问题**：小美是一所中学的信息科技老师，她有一张 seat座位表，平时用来储存学生名字和与他们相对应的座位 id。
其中纵列的 id是连续递增的。小美想改变相邻俩学生的座位，如果学生人数是奇数，则不需要改变最后一个同学的座位。

![image](https://user-images.githubusercontent.com/83053244/192276619-a6d41151-656c-405a-9480-ec8af17f4805.png)

**分析思路**：
这题本质上是交换奇数偶数，奇数+1，偶数-1。这种根据不同分支得到不同列值的任务，可以使用case when 实现。

**代码**：
```sql
#创建表
CREATE TABLE seat ( id INT NOT NULL, student VARCHAR ( 100 ) NOT NULL );
INSERT INTO seat
VALUES
	( 1, 'Abbot' ),
	( 2, 'Doris' ),
	( 3, 'Emerson' ),
	( 4, 'Green' ),
	( 5, 'Jeames' );
```
```sql
#法一
SELECT
	RANK() over ( ORDER BY new_id ) AS id,
	student 
FROM
	(
	SELECT
		student,
	CASE WHEN id % 2 = 1 THEN
	     id + 1 
	     WHEN id % 2 = 0 THEN
   	     id - 1 ELSE NULL 
	END AS new_id 
	FROM
		seat 
	) AS p1;
```
```sql
#法二
SELECT CASE WHEN ( id % 2 = 1 AND id !=( SELECT count(*) FROM seat )) THEN
	    id + 1 
	    WHEN id % 2 = 0 THEN
	    id - 1 ELSE id 
	END AS id,
	student 
FROM
	seat 
ORDER BY
	id;
```
**代码注意点**：
法一和法二的区别是对最后一个奇数学生的处理。法一是让奇数学生id+1，然后再用rank()函数排序，那这个最后学生的id肯定还是会变成原先的id。
法二是直接让这个最后学生的id保持不变，只有奇数且不是最后一个学生才能id+1。

**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192233978-90a68a94-c35b-4a91-a9c4-acb0eca452c3.png)

## 3.分数排名（难度：简单）
**问题**：编写一个 SQL查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

![image](https://user-images.githubusercontent.com/83053244/192276684-4fdfa024-c5f2-44be-b052-59ed808c42e3.png)


**分析思路**：分数排名不应该有间隔，想当然地用到了dense_rank()函数。

**代码**：
```sql
#创建表
CREATE TABLE score (
	id INT,
Score FLOAT ( 3, 2 ));
INSERT INTO score
VALUES
	( 1, 3.50 ),
	( 2, 3.65 ),
	( 3, 4.00 ),
	( 4, 3.85 ),
	( 5, 4.00 ),
	( 6, 3.65 );
```
```sql
SELECT
	Score,
	DENSE_RANK() over ( ORDER BY Score DESC ) AS 'Rank' 
FROM
	score;
```
**代码注意点**：1.rank是sql中的特定名，所以想要用它做别名必须加上引号‘rank’。2.分数越高排名越靠前，所以是降序，用DESC。

**输出结果**：

![image](https://user-images.githubusercontent.com/83053244/192235412-012986b1-c0a3-4f70-86ad-c7f39634ce9b.png)

## 4.连续出现的数字（难度：简单）
**问题**：编写一个 SQL查询，查找所有至少连续出现四次的数字。

![image](https://user-images.githubusercontent.com/83053244/192276747-e3f52a1f-1aea-4e5e-888a-54076729aee7.png)


**分析思路**：查看出现次数，那么必然要使用到count()函数。不同的数字出现次数不一样，所以要group by之后再统计。

**代码**：
```sql
SELECT
	num AS ConsecutiveNums 
FROM
	logs 
GROUP BY
	num 
HAVING
	count(*) > 3;
```
**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192237741-c75c8486-928f-40b0-a721-e9ae29afd634.png)

## 5.树节点（难度：中等）
**问题**：对于 tree表，id是树节点的标识，p_id是其父节点的 id。写一条查询语句打印节点 id及对应的节点类型。按照节点 id排序。注意：如果一个树只有一个节点，只需要输出根节点属性。

![image](https://user-images.githubusercontent.com/83053244/192259719-67435cd6-a27e-4bc9-a76f-0b5919b4e2f4.png)

**分析思路**：分类问题，所以想当然想到case when语句。三种节点的最大区别就是自身是不是父节点，自己的父节点是不是null。

**代码**：
```sql
SELECT
	id,
	CASE WHEN p_id IS NULL THEN
	     'root' 
	     WHEN p_id IS NOT NULL 
             AND id IN ( SELECT p_id FROM tree WHERE p_id IS NOT NULL ) THEN
   	     'inner' 
	     WHEN id NOT IN ( SELECT p_id FROM tree WHERE p_id IS NOT NULL ) THEN
	     'leaf' ELSE NULL 
	END AS type 
FROM
	tree;
```
**代码注意点**：
```sql
#错误判断条件
p_id = NULL 
p_id <> NULL
#正确判断条件
p_id IS NULL
p_id IS NOT NULL 
```
**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192260537-71fe1de5-4dac-4941-91c0-28aa43f625b1.png)

## 6.至少有五名直接下属的经理（难度：中等）
**问题**：每位员工都有一个 Id，并且还有一个对应主管的 Id（Man-agerId）。写一条 SQL语句找出有大于等于5个下属的主管。

![image](https://user-images.githubusercontent.com/83053244/192264975-ec513117-9984-4e98-b7a7-37ebfb690ebc.png)

**分析思路**：首先，我们要先筛选出下属>=5的主管序号（managerId）。然后根据managerId找到对应的姓名。这里可以用到表自联结实现。

**代码**：
```sql
SELECT
	p1.NAME 
FROM
	employee AS p1
	INNER JOIN ( SELECT managerId, count(*) AS subordinate FROM employee GROUP BY managerId HAVING subordinate >= 5 ) AS p2 
WHERE
	p1.id = p2.managerId;
```
**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192265425-7d2fdca2-e015-4889-884c-82db22014fe6.png)

## 7.分数排名（难度：简单）
**问题**：练习3的分数表，实现排名功能，但是排名需要是非连续的

**代码**：
```sql
SELECT
	Score,
	RANK() over ( ORDER BY Score DESC ) AS 'Rank' 
FROM
	score;
```
**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192266076-0038d177-5a9a-4cb4-bb37-8e7a03d02e2c.png)

## 8.查询回答率最高的问题（难度：中等）
**问题**：survey_log表如下，uid是用户 id；action的值为：“show” ，“answer” ，“skip” ；当 action是"answer"时，answer_id不为空，
相反，当 action是"show"和"skip"时为空（null）；q_num是问题的数字序号。写一条 sql语句找出回答率最高的问题。
**最高回答率的意思是：同一个问题回答次数占出现次数的比例。**

![image](https://user-images.githubusercontent.com/83053244/192274803-66cda70a-9487-49ee-85df-b744c26c7a93.png)

**分析思路**：首先计算每个问题的回答率，就想到了用group by结合count()来实现。然后按照回答率降序排序。最后选择第一行的question_id也就是回答率最高的问题。

**代码**：
```sql
SELECT
	question_id as survey_log
FROM
	( SELECT 
		question_id, count( answer_id )/ count( question_id ) AS ratio 
	  FROM 
	  	survey_log
	  GROUP BY 
		question_id 
	  ORDER BY
		ratio DESC ) AS p 
	LIMIT 1;
```

**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192275826-e3a4f503-89c4-4770-897b-6c1969ff248b.png)

## 9.各部门前 3 高工资的员工（难度：中等）
**问题**：将项目一中的 employee表清空，重新创建如下新表，并保留department表。编写一个 SQL查询，找出每个部门工资前三高的员工。

![image](https://user-images.githubusercontent.com/83053244/192410048-aca0bc2a-52ab-4323-aea5-5c289041e288.png)
![image](https://user-images.githubusercontent.com/83053244/192410070-c10a50d7-33da-4e1e-8644-5fd1b1c0ef26.png)


**分析思路**：刚开始我想当然地用group by去分组，找出每组前三的工资即可，但是发现事情没这么简单。因为group by子句只具备分组汇总功能，每组总是只能保留一行聚合后的结果，根本无法每组保留三行结果。后来我想到了窗口函数中有partition by子句有类似分组功能。到这里，我才真正搞清楚group by 和 partition by 的区别。**GROUP BY 子句只具备分组汇总功能，而PARTITION BY 子句虽然不具备 GROUP BY 子句的汇总功能，但是它不会改变原始表中记录的行数，想保留多少行都可以。**

**代码**：
```sql
SELECT
	d.site AS Department,
	e.Employee,
	e.Salary 
FROM
	( SELECT 
		DepartmentId,
		NAME AS Employee, 
		Salary, 
		rank() over ( PARTITION BY DepartmentId ORDER BY Salary DESC ) AS ranking 
	  FROM 
		employee ) AS e
	INNER JOIN department AS d ON e.DepartmentId = d.Id 
WHERE
	ranking <= 3;
```

**小结**：

如果遇到**按类别分组**时，有两种选择即 group by 和 partition by，需要汇总功能用group by，需要排序等保留原始记录用partition by。

如果遇到**按条件分组**时，就选择case when，比如 > < = 这种。

**输出结果**：

![image](https://user-images.githubusercontent.com/83053244/192411419-b1bd7fe4-a68b-4af7-aee4-ee9f395b18b1.png)

## 10.平面上最近距离 (难度:困难）
**问题**：point_2d表包含一个平面内一些点（超过两个）的坐标值（x，y）。写一条查询语句求出这些点中的最短距离并保留 2位小数。

![image](https://user-images.githubusercontent.com/83053244/192420670-7e45be80-2ce4-46e7-9467-1e9e29495ee1.png)


**分析思路**：首先我想到的就是表自联结，表1的第一个点和表2所有点求距离，然后是表1的第二个点和表2所有点求距离，依次类推。这样就能求出所有点之间的距离了。然后按照距离排序选择最短的距离就好了。

**代码**：
```sql
#方法一：更推荐法一，代码更精炼一点
SELECT
	* 
FROM
	(
	SELECT
		p1.x AS x1,
		p1.y AS y1,
		p2.x AS x2,
		p2.y AS y2,
		cast(
			sqrt(
			power(( p1.x - p2.x ), 2 )+ power(( p1.y - p2.y ), 2 )) AS DECIMAL ( 10, 2 )) AS distance 
	FROM
		point_2d AS p1
		LEFT OUTER JOIN point_2d AS p2 ON sqrt(
		power(( p1.x - p2.x ), 2 )+ power(( p1.y - p2.y ), 2 )) <> 0 
	) AS p3 
ORDER BY
	distance 
	LIMIT 1;
```
```sql
#方法二：
SELECT
	* 
FROM
	(
	SELECT
		*,
		rank() over ( ORDER BY distance ) AS ranking 
	FROM
		(
		SELECT
			p1.x AS x1,
			p1.y AS y1,
			p2.x AS x2,
			p2.y AS y2,
			cast(
				sqrt(
				power(( p1.x - p2.x ), 2 )+ power(( p1.y - p2.y ), 2 )) AS DECIMAL ( 10, 2 )) AS distance 
		FROM
			point_2d AS p1
			LEFT OUTER JOIN point_2d AS p2 ON sqrt(power(( p1.x - p2.x ), 2 )+ power(( p1.y - p2.y ), 2 )) <> 0 
		) AS p3 
	) AS p4 
WHERE
	p4.ranking = 1;
```

**代码注意点**：

`cast(number as decimal(10,2))`，可以实现保留两位有效数字。`sqrt(power(( p1.x - p2.x ), 2 )+ power(( p1.y - p2.y ), 2 ))`是距离公式。

**where中不能使用max() 等聚合函数，rank() over 等窗口函数只能在 SELECT 子句中使用，order by 等在select前执行的子句不能使用select定义的别名！**违反这些规则会报错，而且不记清楚会浪费很多时间。

**小结**：法一和法二区别在于算出点与点之间的距离之后（如下表），怎么找到最短距离。最精炼的办法（法一）就是在新表的基础上，按照distance order by排序，然后选择第一行就可以了。

![image](https://user-images.githubusercontent.com/83053244/192419133-9124d8e7-fdd5-4d8d-a274-8815b73f5f7d.png)

**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192420611-ea265c9c-78c3-464c-b46e-c4bf06b6a676.png)

## 11.行程和用户（难度：困难）
**问题**：order表中存所有出租车的行程信息。每段行程有唯一键 Id，Client_Id和 Driver_Id是 Users表中 Users_Id
的外键。Status是枚举类型，枚举成员为 (‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’)。

![image](https://user-images.githubusercontent.com/83053244/192486758-a96f129e-eed8-451e-a88d-431cc0b411a9.png)

Users表存所有用户。每个用户有唯一键 Users_Id。Banned表示这个用户是否被禁止，Role则是一个表示（‘client’, ‘driver’, ‘partner’ ）的枚举类型。

![image](https://user-images.githubusercontent.com/83053244/192486883-bfd16b1c-abe4-42b6-939e-4cd7ffe03bb3.png)

写一段 SQL语句查出每天非禁止用户的取消率。取消率（Cancellation Rate）：取消次数/总次数，保留两位小数。

**分析思路**：这个题目有两种理解，第二种放在11.2，这里只讲第一种理解。只要订单有违禁用户，那么就不看这条订单。所以剩下的订单肯定都是正常用户。**我们只需要取出只包含正常用户的数据作为子表，然后计算每天取消率就行了**。怎么取出子表呢？先取出 users表中 banned 是 yes 的 users_id,然后只要 order表中的 client_id 和 driver_id 不在这个 users_id 集合中的订单，就是可以用的订单。 

**代码**：
```sql
#创建主表users
CREATE TABLE users ( users_id INT PRIMARY KEY, banned enum ( 'No', 'Yes' ) NOT NULL, role enum ( 'client', 'driver', 'partner' ) NOT NULL );
INSERT INTO users
VALUES
	( 1, 1, 1 ),
	( 2, 2, 1 ),
	( 3, 1, 1 ),
	( 4, 1, 1 ),
	( 10, 1, 2 ),
	( 11, 1, 2 ),
	( 12, 1, 2 ),
	( 13, 1, 2 );

#创建从表
CREATE TABLE `order` (
	id INT PRIMARY KEY auto_increment,
	client_id INT REFERENCES users ( users_id ),
	driver_id INT REFERENCES users ( users_id ),
	city_id INT,
	`status` enum ( 'completed', 'cancelled_by_driver', 'cancelled_by_client' ),
	request_at date 
);
INSERT INTO `order` ( client_id, driver_id, city_id, STATUS, request_at )
VALUES
	( 1, 10, 1, 1, '2013-10-1' ),
	( 2, 11, 1, 2, '2013-10-1' ),
	( 3, 12, 6, 1, '2013-10-1' ),
	( 4, 13, 6, 3, '2013-10-1' ),
	( 1, 10, 1, 1, '2013-10-2' ),
	( 2, 11, 6, 1, '2013-10-2' ),
	( 3, 12, 6, 1, '2013-10-2' ),
	( 2, 12, 12, 1, '2013-10-3' ),
	( 3, 10, 12, 1, '2013-10-3' ),
	( 4, 13, 12, 2, '2013-10-3' );
```
```sql
SELECT
	request_at AS `date`,
	cast(
	count( CASE WHEN `status` <> 'completed' THEN 1 END )/ count( `status` ) AS DECIMAL ( 10, 2 )) AS 'Cancellation Rate' 
FROM
	(
	SELECT
		* 
	FROM
		`order` 
	WHERE
		( client_id NOT IN ( SELECT users_id FROM users WHERE banned = 'Yes' ) ) 
		AND (
		driver_id NOT IN ( SELECT users_id FROM users WHERE banned = 'Yes' )) 
	) AS p1 
GROUP BY
	request_at;
```

**代码注意点**：`order`是 SQL 中的关键字，用作表名时需要使用反引号包围。p1表如下（只有正常用户）：

![image](https://user-images.githubusercontent.com/83053244/192490343-e2b476c6-c35c-4462-81e2-96a5d4e6bb59.png)


**小结**：
1. 主键（primary key）和唯一键（unique key）区别：主键不能重复, 不能为空。唯一键是为了避免添加重复数据。**一张表中只能有一个主键, 但是可以有多个唯一键**。

2. 自增长字段（AUTO_INCREMENT）：默认下，AUTO_INCREMENT 的初始值是 1，每新增一条记录，字段值自动加 1.**auto_increment的字段必须是主键, 但是主键不一定是auto_increment的**。

3. 外键（Foreign Key）：用于将两个表连接在一起，让两个表的数据保持同步。**一个表的外键用来指向另一个表的主键（Primary Key）**。包含外键的表称为从表，被指向的表称为主表。所以order表是从表，users表是主表。从表的数据受到主表的约束，向从表中插入或者更新数据时，外键的值必须存在于主表的主键中。参考：http://c.biancheng.net/sql/foreign-key.html

4.枚举类型（ENUM）：使用数字索引(1，2，3，…)来表示字符串值。例如：`priority ENUM('Low', 'Medium', 'High') NOT NULL`。参考：https://www.yiibai.com/mysql/enum.html

5.where 中可以用in，但是in 后面不能接表名，只能接集合或者select子句。**这题用的两个 where 很精妙，值得多看看。**

**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192490545-6c9a3247-e1be-4310-9f27-82a5e7de7568.png)

## 11.2 行程和用户（难度：困难）
**分析思路2**：只有违禁用户取消的订单才需要排除，违禁用户坐在车上的订单是可以用的。**那么我们先找出取消者cancel_id（order2表），然后比对出这些取消者是不是违禁用户（order3表）**。最后计算取消率。

**代码**：
```sql
SELECT
	request_at AS DAY,
	cast(
	count( CASE WHEN cancel_id <> 0 THEN 1 END )/ count( cancel_id ) AS DECIMAL ( 10, 2 )) AS 'Cancellation Rate' 
FROM
	(
	SELECT
		order2.id,
		order2.cancel_id,
		order2.request_at,
		users.banned 
	FROM
		( SELECT *, CASE WHEN STATUS = 2 THEN driver_id WHEN STATUS = 3 THEN client_id ELSE 0 END AS cancel_id FROM `order` ) AS order2
		LEFT OUTER JOIN users ON order2.cancel_id = users.users_id 
	) AS order3 
GROUP BY
	request_at;
```

**代码注意点**：order2表，order3表

![image](https://user-images.githubusercontent.com/83053244/192493282-b390b792-740d-4a04-bdcc-1de36bbb20af.png)

![image](https://user-images.githubusercontent.com/83053244/192493452-58419d3e-bebc-42e5-a380-bc107140f128.png)

**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192494138-18d1b5f6-92de-4cae-ab97-4051c7387bd1.png)

## 12.行转列（难度：中等）
**问题**：假设 A B C三位小朋友期末考试成绩如下所示

![image](https://user-images.githubusercontent.com/83053244/192494584-eba67ef9-4781-44e8-b2a3-b6d6f95d24d3.png)

请使用 SQL代码将以上成绩转换为如下格式：

![image](https://user-images.githubusercontent.com/83053244/192494634-4641dca7-2bd3-4b0f-9a0a-d82e89ef0f56.png)

**代码**：
```sql
#case when 与聚合函数的结合应用
SELECT
	`name`,
	sum( CASE WHEN subject = 1 THEN score ELSE NULL END ) AS chinese,
	sum( CASE WHEN subject = 2 THEN score ELSE NULL END ) AS math,
	sum( CASE WHEN subject = 3 THEN score ELSE NULL END ) AS english 
FROM
	quiz 
GROUP BY
	`name`;
```

## 13.列转行（难度：中等）
**问题**：上面12题的反向操作

**代码**：
```sql
#将12题的查询结果建成新表quiz2
CREATE TABLE quiz2 (
	SELECT
		`name`,
		sum( CASE WHEN subject = 1 THEN score ELSE NULL END ) AS chinese,
		sum( CASE WHEN subject = 2 THEN score ELSE NULL END ) AS math,
		sum( CASE WHEN subject = 3 THEN score ELSE NULL END ) AS english 
	FROM
		quiz 
	GROUP BY
		`name` 
	);
	
#反向操作
(select `name`,'chinese' as subject, chinese as score from quiz2
union
select `name`,'math' as subject, math as score from quiz2
union
select `name`,'english' as subject, english as score from quiz2
) order by `name`;
```

## 14.带货主播(难度：困难）
**问题**：假设，某平台 2021年主播带货销售额日统计数据如下，表名 anchor_sales。

![image](https://user-images.githubusercontent.com/83053244/192496766-49b76498-a4bd-4c5a-8637-cd09c3dbd957.png)

如果某主播的某日销售额占比达到该平台当日销售总额的 90% 及以上，则称该主播为明星主播，当天也称为明星主播日。请使用 SQL完成如下计算：
a. 2021年有多少个明星主播日？
b. 2021年有多少个明星主播？

**分析思路**：首先算出每天的总销售额，然后采用自联结将每天总销售额加到每一条记录中。然后计算每一条记录是不是star，得到表p3。最后挑出 star 是 yes 的记录，采用`count(distinct 列名)`计算出现的主播和主播日次数，避免了重复计算。p3如下：

![image](https://user-images.githubusercontent.com/83053244/192498379-4e635776-adba-4129-a69a-263444076c04.png)

**代码**：
```sql
SELECT
	count( DISTINCT date ) AS star_date,
	count( DISTINCT anchor_name ) AS star_anchor 
FROM
	(
	SELECT
		p1.anchor_name,
		p1.date,
	CASE
			
			WHEN sales / sum_sales >= 0.9 THEN
			'yes' ELSE 'no' 
		END AS star 
	FROM
		anchor_sales AS p1
		INNER JOIN ( SELECT `date`, sum( sales ) AS sum_sales FROM anchor_sales GROUP BY `date` ) AS p2 ON p1.date = p2.date 
	) AS p3 
WHERE
	star = 'yes';
```

**运行结果**：

![image](https://user-images.githubusercontent.com/83053244/192498536-3835c2c1-6e9d-447f-99d3-499b7edc7c72.png)

## 15.查看执行计划
MySQL中如何查看 sql语句的执行计划？可以看到哪些信息？

使用**explain关键字**可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的，分析你的查询语句或是表结构的性能瓶颈。
具体知识参考：https://blog.csdn.net/weixin_41558728/article/details/81704916

## 16.解释 ACID
解释一下 SQL数据库中 ACID是指什么？

**事务的基本要素（ACID）：**

　　1、原子性（Atomicity）：**事务开始后所有操作，要么全部做完，要么全部不做**，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。

　　 2、一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。其实一致性也是因为原子型的一种表现

　　 3、隔离性（Isolation）：**同一时间，只允许一个事务请求同一数据**，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。串行化

　　 4、持久性（Durability）：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。
   
   更详细参考：（有事务并发问题/事务隔离级别等）https://www.yisu.com/zixun/453098.html
   
## 17.行转列2（困难：中等）
**问题**：假设有如下比赛结果：

![image](https://user-images.githubusercontent.com/83053244/192500294-53065a32-bf6a-4237-92a2-36e54ab1dc26.png)

请使用 SQL将比赛结果转换为如下形式：

![image](https://user-images.githubusercontent.com/83053244/192500359-b79cb9ec-bb8a-449e-8d1a-5c5aa737bb9e.png)


















