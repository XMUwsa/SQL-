## 1.各部门工资最高的员工（难度：中等）
**问题**：创建两个表。Employee表，包含所有员工信息，每个员工有其对应的 Id, salary和 department Id。department表，包含部门号和部门名。然后找出每个部门工资最高的员工。

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

**分析思路**：查看出现次数，那么必然要使用到count()函数。不同的数字出现次数不一样，所以要group by之后再统计。

**代码**：
```sql
SELECT
	num AS ConsecutiveNums 
FROM
LOGS 
GROUP BY
	num 
HAVING
	count(*) > 3;
```
**运行结果**：
![image](https://user-images.githubusercontent.com/83053244/192237741-c75c8486-928f-40b0-a721-e9ae29afd634.png)

## 5.




