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







