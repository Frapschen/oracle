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
已启用自动跟踪
显示执行计划以及语句的统计信息。

DEPARTMENT_NAME                     部门总人数       平均工资
------------------------------ ---------- ----------
IT                                      5       5760
Sales                                  34 8955.88235

Explain Plan
-----------------------------------------------------------

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
--------------------------------------
Plan hash value: 3073107731
 
------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name           | Rows  | Bytes | Cost (%CPU)| Time     |
------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                |     1 |    23 |     5  (20)| 00:00:01 |
|   1 |  HASH GROUP BY                |                |     1 |    23 |     5  (20)| 00:00:01 |
|   2 |   NESTED LOOPS                |                |    19 |   437 |     4   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                |    20 |   437 |     4   (0)| 00:00:01 |
|*  4 |     TABLE ACCESS FULL         | DEPARTMENTS    |     2 |    32 |     3   (0)| 00:00:01 |
|*  5 |     INDEX RANGE SCAN          | IDX$$_030E0001 |    10 |       |     0   (0)| 00:00:01 |

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
--------------------------------------
|   6 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES      |    10 |    70 |     1   (0)| 00:00:01 |
------------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   4 - filter("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   5 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Note
-----

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
   - this is an adaptive plan

Statistics
-----------------------------------------------------------
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               4  buffer is not pinned count
              77  buffer is pinned count
             666  bytes received via SQL*Net from client
           44800  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               4  calls to kcmgcs
              10  consistent gets
              10  consistent gets from cache
              10  consistent gets pin
              10  consistent gets pin (fastpath)
               2  execute count
               2  index scans kdiixs1
           81920  logical read bytes from cache
               7  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
              10  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
              39  table fetch by rowid
               5  table scan blocks gotten
              27  table scan disk non-IMC rows gotten
              27  table scan rows gotten
               1  table scans (short tables)
              26  user calls
```
我们看到Cost=2,Rows=20,在谓词信息中有一次索引搜索access,一次全表搜索,consistent gets=10.
              
### 执行查询语句2：
```
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```
执行结果：
```
已启用自动跟踪
显示执行计划以及语句的统计信息。

DEPARTMENT_NAME                     部门总人数       平均工资
------------------------------ ---------- ----------
IT                                      5       5760
Sales                                  34 8955.88235

Explain Plan
-----------------------------------------------------------

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
--------------------------------------
Plan hash value: 2128232041
 
----------------------------------------------------------------------------------------------
| Id  | Operation                      | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT               |             |     1 |    23 |     7  (29)| 00:00:01 |
|*  1 |  FILTER                        |             |       |       |            |          |
|   2 |   HASH GROUP BY                |             |     1 |    23 |     7  (29)| 00:00:01 |
|   3 |    MERGE JOIN                  |             |   106 |  2438 |     6  (17)| 00:00:01 |
|   4 |     TABLE ACCESS BY INDEX ROWID| DEPARTMENTS |    27 |   432 |     2   (0)| 00:00:01 |
|   5 |      INDEX FULL SCAN           | DEPT_ID_PK  |    27 |       |     1   (0)| 00:00:01 |

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
--------------------------------------
|*  6 |     SORT JOIN                  |             |   107 |   749 |     4  (25)| 00:00:01 |
|   7 |      TABLE ACCESS FULL         | EMPLOYEES   |   107 |   749 |     3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter("DEPARTMENT_NAME"='IT' OR "DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
       filter("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

Statistics
-----------------------------------------------------------
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               3  buffer is not pinned count
              52  buffer is pinned count
             669  bytes received via SQL*Net from client
           44804  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               4  calls to kcmgcs
               9  consistent gets
               9  consistent gets from cache
               9  consistent gets pin
               9  consistent gets pin (fastpath)
               1  cursor authentications
               2  execute count
               1  index scans kdiixs1
           73728  logical read bytes from cache
               6  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
               1  session cursor cache hits
               9  session logical reads
               2  sorts (memory)
            1285  sorts (rows)
              27  table fetch by rowid
               5  table scan blocks gotten
             107  table scan disk non-IMC rows gotten
             107  table scan rows gotten
               1  table scans (short tables)
              26  user calls

```
我们看到，Cost=5,Rows=107,在在谓词信息中有一次索引搜索access，一次全表搜索filter,consitent gets=8。对比之下，查询语句1更优。
下面运行sqldeverloper:
![img](https://github.com/Frapschen/oracle/blob/master/test1/sqldeveloper_result.png)
没有提供更加优化的sql.

