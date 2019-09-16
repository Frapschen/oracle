## 试验一
### 执行查询语句1：
```
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
```
结果：
![Image text](https://github.com/Frapschen/oracle/blob/master/test1/%E7%BB%93%E6%9E%9C1.png)
### 执行查询语句2：
```
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```
结果：
![Image text](https://github.com/Frapschen/oracle/blob/master/test1/%E7%BB%93%E6%9E%9C2.png)
