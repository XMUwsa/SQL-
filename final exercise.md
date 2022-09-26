## 1.各部门工资最高的员工（难度：中等）
**问题**：创建两个表。Employee表，包含所有员工信息，每个员工有其对应的 Id, salary和 department Id。department表，包含部门号和部门名。然后找出每个部门工资最高的员工。

**分析思路**：先找出每个部门最高工资是多少；然后按照部门id将最高工资作为新的一列添加到employee表，并筛选出salary与max_salary相等的行；最后将筛选结果与department表联结。本题考查了两点：自联结和多表联结

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
**输出结果**：

![image](https://user-images.githubusercontent.com/83053244/192179907-bf6fa58b-af1d-4839-826c-fa35d5d17e3e.png)

## 2.换座位（难度：中等）

