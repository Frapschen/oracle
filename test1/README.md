# 试验一
## 书上实验
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
我们看到，Cost=7,Rows=106,在在谓词信息中有一次索引搜索access，两次全表搜索filter,consitent gets=8。对比之下，查询语句1更优。
下面运行sqldeverloper:
![img](https://github.com/Frapschen/oracle/blob/master/test1/sqldeveloper_result_before.png)
我们看到sqldeverloper给出了优化指导，它让我们建立索引：
```
Recommendation (estimated benefit: 59.99%)
  ------------------------------------------
  - 考虑运行可以改进物理方案设计的访问指导或者创建推荐的索引。
    create index HR.IDX$$_03390001 on HR.DEPARTMENTS("DEPARTMENT_NAME","DEPARTM
    ENT_ID");

  Rationale
  ---------
    创建推荐的索引可以显著地改进此语句的执行计划。但是, 使用典型的 SQL 工作量运行 "访问指导"
    可能比单个语句更可取。通过这种方法可以获得全面的索引建议案, 包括计算索引维护的开销和附加的空间消耗。

```
建立索引，再次执行SQL语句1:
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

Plan hash value: 3939821256
 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   1 |  SORT GROUP BY NOSORT         |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     2   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     2   (0)| 00:00:01 |
|   4 |     INLIST ITERATOR           |                   |       |       |            |          |
|*  5 |      INDEX RANGE SCAN         | INDEX1            |     2 |    32 |     1   (0)| 00:00:01 |

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
|*  6 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   7 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   5 - access("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")

Statistics
-----------------------------------------------------------
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               6  buffer is not pinned count
              76  buffer is pinned count
             666  bytes received via SQL*Net from client
           47227  bytes sent via SQL*Net to client
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
               5  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
              39  table fetch by rowid
              26  user calls

```
现在发现Cost=2,Rows=20,在谓词信息中有两次索引搜索access,,consistent gets=5.相比于没有建立索引前，性能得到了提升。再次运行优化指导，发现没有优化建议了，如图：
![img](https://github.com/Frapschen/oracle/blob/master/test1/sqldeveloper_result_after.png)
## 我自己构建的SQL语句
```
SET AUTOTRACE ON
select avg(e.salary) as "工资",j.country_name from 
employees e,(select x.department_id, c.country_name  from 
(select d.department_id,l.country_id from departments d, locations l where d.location_id=l.location_id) x, countries c
where x.country_id=c.country_id) j
where e.department_id=j.department_id
GROUP by j.country_name
```
查询结果：
```
已启用自动跟踪
显示执行计划以及语句的统计信息。

        工资 COUNTRY_NAME                            
---------- ----------------------------------------
8885.71429 United Kingdom                          
5067.88235 United States of America                
     10000 Germany                                 
      9500 Canada                                  

Explain Plan
-----------------------------------------------------------

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           

Plan hash value: 2167937235
 
-------------------------------------------------------------------------------------------------
| Id  | Operation                  | Name               | Rows  | Bytes | Cost (%CPU)| Time     |
-------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT           |                    |   106 |  6042 |     8  (13)| 00:00:01 |
|   1 |  HASH GROUP BY             |                    |   106 |  6042 |     8  (13)| 00:00:01 |
|*  2 |   HASH JOIN                |                    |   106 |  6042 |     7   (0)| 00:00:01 |
|*  3 |    HASH JOIN               |                    |    27 |  1350 |     4   (0)| 00:00:01 |
|   4 |     NESTED LOOPS           |                    |    23 |   989 |     2   (0)| 00:00:01 |
|   5 |      VIEW                  | index$_join$_005   |    23 |   391 |     2   (0)| 00:00:01 |

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           

|*  6 |       HASH JOIN            |                    |       |       |            |          |
|   7 |        INDEX FAST FULL SCAN| LOC_COUNTRY_IX     |    23 |   391 |     1   (0)| 00:00:01 |
|   8 |        INDEX FAST FULL SCAN| LOC_ID_PK          |    23 |   391 |     1   (0)| 00:00:01 |
|*  9 |      INDEX UNIQUE SCAN     | COUNTRY_C_ID_PK    |     1 |    26 |     0   (0)| 00:00:01 |
|  10 |     VIEW                   | index$_join$_004   |    27 |   189 |     2   (0)| 00:00:01 |
|* 11 |      HASH JOIN             |                    |       |       |            |          |
|  12 |       INDEX FAST FULL SCAN | DEPARTMENTS_INDEX1 |    27 |   189 |     1   (0)| 00:00:01 |
|  13 |       INDEX FAST FULL SCAN | DEPT_LOCATION_IX   |    27 |   189 |     1   (0)| 00:00:01 |
|  14 |    TABLE ACCESS FULL       | EMPLOYEES          |   107 |   749 |     3   (0)| 00:00:01 |
-------------------------------------------------------------------------------------------------
 

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           

Predicate Information (identified by operation id):
---------------------------------------------------
 
   2 - access("E"."DEPARTMENT_ID"="D"."DEPARTMENT_ID")
   3 - access("D"."LOCATION_ID"="L"."LOCATION_ID")
   6 - access(ROWID=ROWID)
   9 - access("L"."COUNTRY_ID"="C"."COUNTRY_ID")
  11 - access(ROWID=ROWID)
 
Note
-----

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           

   - dynamic statistics used: dynamic sampling (level=2)
   - this is an adaptive plan

Statistics
-----------------------------------------------------------
               1  CPU used by this session
               1  CPU used when call started
              26  Requests to/from client
              25  SQL*Net roundtrips to/from client
               2  buffer is not pinned count
               3  buffer is pinned count
             757  bytes received via SQL*Net from client
           44808  bytes sent via SQL*Net to client
               2  calls to get snapshot scn: kcmgss
              16  calls to kcmgcs
              27  consistent gets
               2  consistent gets examination
               2  consistent gets examination (fastpath)
              27  consistent gets from cache
              25  consistent gets pin
              25  consistent gets pin (fastpath)
               1  cursor authentications
               2  execute count
               4  index fast full scans (full)
              23  index fetch by key
          221184  logical read bytes from cache
              11  no work - consistent read gets
              25  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               2  parse count (total)
              27  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
               5  table scan blocks gotten
             107  table scan disk non-IMC rows gotten
             107  table scan rows gotten
               1  table scans (short tables)
              26  user calls

```
我们看到Cost=8,Rows=107,在谓词信息中有五次索引搜索access,consistent gets=27
