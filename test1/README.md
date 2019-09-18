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
执行结果：
```
EMPLOYEE_ID FIRST_NAME           MANAGER_ID MANAGER_NAME        
----------- -------------------- ---------- --------------------
        100 Steven中国                                          
        101 Neena                       100 Steven中国          
        102 Lex                         100 Steven中国          
        103 Alexander                   102 Lex     
        ···
已选择 107 行。

Explain Plan
-----------------------------------------------------------

PLAN_TABLE_OUTPUT
-----------------------------------------------------------
Plan hash value: 4265739349

---------------------------------------------------------------------------------------------
| Id  | Operation                   | Name          | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |               |   107 |  1605 |    21   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES     |     1 |    11 |     1   (0)| 00:00:01 |
|*  2 |   INDEX UNIQUE SCAN         | EMP_EMP_ID_PK |     1 |       |     0   (0)| 00:00:01 |
|   3 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES     |   107 |  1605 |     3   (0)| 00:00:01 |
|   4 |   INDEX FULL SCAN           | EMP_EMP_ID_PK |   107 |       |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------

PLAN_TABLE_OUTPUT 
-----------------------------------------------------------

Predicate Information (identified by operation id):
-----------------------------------------------------------

   2 - access("M"."EMPLOYEE_ID"=:B1)
   
Statistics
-----------------------------------------------------------
              27  Requests to/from client
              26  SQL*Net roundtrips to/from client
              44  buffer is not pinned count
             213  buffer is pinned count
             614  bytes received via SQL*Net from client
           49405  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               2  calls to kcmgcs
              27  consistent gets
              19  consistent gets examination
              19  consistent gets examination (fastpath)
              27  consistent gets from cache
               8  consistent gets pin
               8  consistent gets pin (fastpath)
               2  execute count
              18  index fetch by key
               1  index scans kdiixs1
          221184  logical read bytes from cache
               7  no work - consistent read gets
              26  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
              18  rows fetched via callback
               1  session cursor cache hits
              27  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
             125  table fetch by rowid
              27  user calls
```
我们看到consistent gets为27，它不是最有效的语句。
              
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
```

```
