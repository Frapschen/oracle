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
Plan hash value: 872036552
 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   1 |  SORT GROUP BY NOSORT         |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     2   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     2   (0)| 00:00:01 |
|   4 |     INLIST ITERATOR           |                   |       |       |            |          |
|*  5 |      INDEX RANGE SCAN         | IDX$$_01220001    |     2 |    32 |     1   (0)| 00:00:01 |


PLAN_TABLE_OUTPUT 
 -----------------------------------------------------------  
|*  6 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   7 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------


Predicate Information (identified by operation id):
 -----------------------------------------------------------  
 
   5 - access("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")


Statistics
-----------------------------------------------------------
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               6  buffer is not pinned count
              76  buffer is pinned count
             664  bytes received via SQL*Net from client
           47134  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               2  calls to kcmgcs
               5  consistent gets
               5  consistent gets from cache
               5  consistent gets pin
               5  consistent gets pin (fastpath)
               2  execute count
               4  index scans kdiixs1
           40960  logical read bytes from cache
               3  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
              -1  session cursor cache count
               2  session cursor cache hits
               5  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
              39  table fetch by rowid
              26  user calls
```
我们看到Cost=2,Rows=20,在谓词信息中有两次索引搜索access,consistent gets为5.
              
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
-----------------------------------------------------------
Plan hash value: 3765253595

---------------------------------------------------------------------------------------
| Id  | Operation            | Name           | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT     |                |     1 |    23 |     5  (20)| 00:00:01 |
|*  1 |  FILTER              |                |       |       |            |          |
|   2 |   HASH GROUP BY      |                |     1 |    23 |     5  (20)| 00:00:01 |
|*  3 |    HASH JOIN         |                |   106 |  2438 |     4   (0)| 00:00:01 |
|   4 |     INDEX FULL SCAN  | IDX$$_01220001 |    27 |   432 |     1   (0)| 00:00:01 |
|   5 |     TABLE ACCESS FULL| EMPLOYEES      |   107 |   749 |     3   (0)| 00:00:01 |

PLAN_TABLE_OUTPUT
-----------------------------------------------------------

Predicate Information (identified by operation id):
-----------------------------------------------------------

   1 - filter("DEPARTMENT_NAME"='IT' OR "DEPARTMENT_NAME"='Sales')
   3 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

Statistics
-----------------------------------------------------------
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               1  buffer is not pinned count
             669  bytes received via SQL*Net from client
           47164  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
               4  calls to kcmgcs
               8  consistent gets
               8  consistent gets from cache
               8  consistent gets pin
               8  consistent gets pin (fastpath)
               2  execute count
               1  index scans kdiixs1
           65536  logical read bytes from cache
               5  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
              -3  session cursor cache count
               1  session cursor cache hits
               8  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
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

