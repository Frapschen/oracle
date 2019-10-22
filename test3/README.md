# 试验一
首先在system用户下给我的的用户授权表空间,结果如下：
```
User CMQ_USER已变更。
User CMQ_USER已变更。
User CMQ_USER已变更。
```
然后登陆我的用户cmq_user，创建orders（订单表）和order_details（订单详表），两个表通过列order_id建立主外键关联。orders表按范围分区进行存储，order_details使用引用分区进行存储。建表语句如下：
```
declare
      num   number;
begin
      select count(1) into num from user_tables where TABLE_NAME = 'ORDER_DETAILS';
      if   num=1   then
          execute immediate 'drop table ORDER_DETAILS cascade constraints PURGE';
      end   if;

      select count(1) into num from user_tables where TABLE_NAME = 'ORDERS';
      if   num=1   then
          execute immediate 'drop table ORDERS cascade constraints PURGE';
      end   if;
end;
/

CREATE TABLE ORDERS
(
  ORDER_ID NUMBER(10, 0) NOT NULL
, CUSTOMER_NAME VARCHAR2(40 BYTE) NOT NULL
, CUSTOMER_TEL VARCHAR2(40 BYTE) NOT NULL
, ORDER_DATE DATE NOT NULL
, EMPLOYEE_ID NUMBER(6, 0) NOT NULL
, DISCOUNT NUMBER(8, 2) DEFAULT 0
, TRADE_RECEIVABLE NUMBER(8, 2) DEFAULT 0
, CONSTRAINT ORDERS_PK PRIMARY KEY
  (
    ORDER_ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX ORDERS_PK ON ORDERS (ORDER_ID ASC)
      LOGGING
      TABLESPACE USERS
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
TABLESPACE USERS
PCTFREE 10
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NOPARALLEL
PARTITION BY RANGE (ORDER_DATE)
(
  PARTITION PARTITION_2015 VALUES LESS THAN (TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_2016 VALUES LESS THAN (TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_2017 VALUES LESS THAN (TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_2018 VALUES LESS THAN (TO_DATE(' 2019-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS02
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_2019 VALUES LESS THAN (TO_DATE(' 2020-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS02
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_2020 VALUES LESS THAN (TO_DATE(' 2021-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS02
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_2021 VALUES LESS THAN (TO_DATE(' 2022-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS03
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
);

CREATE TABLE order_details
(
id NUMBER(10, 0) NOT NULL
, order_id NUMBER(10, 0) NOT NULL
, product_name VARCHAR2(40 BYTE) NOT NULL
, product_num NUMBER(8, 2) NOT NULL
, product_price NUMBER(8, 2) NOT NULL
, CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)
REFERENCES orders  (  order_id   )
ENABLE
)
TABLESPACE USERS
PCTFREE 10 INITRANS 1
STORAGE (BUFFER_POOL DEFAULT )
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1);

declare
  dt date;
  m number(8,2);
  V_EMPLOYEE_ID NUMBER(6);
  v_order_id number(10);
  v_name varchar2(100);
  v_tel varchar2(100);
  v number(10,2);
  v_order_detail_id number;
begin
/*
system login:
ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS;
ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS02;
ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS03;
*/
  v_order_detail_id:=1;
  delete from order_details;
  delete from orders;
  for i in 1..10000
  loop
    if i mod 6 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2015
    elsif i mod 6 =1 then
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2016
    elsif i mod 6 =2 then
      dt:=to_date('2017-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2017
    elsif i mod 6 =3 then
      dt:=to_date('2018-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2018
    elsif i mod 6 =4 then
      dt:=to_date('2019-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2019
    else
      dt:=to_date('2020-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2020
    end if;
    V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
    --插入订单
    v_order_id:=i;
    v_name := 'aa'|| 'aa';
    v_name := 'zhang' || i;
    v_tel := '139888883' || i;
    insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
      values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
    --插入订单y一个订单包括3个产品
    v:=dbms_random.value(10000,4000);
    v_name:='computer'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (v_order_detail_id,v_order_id,v_name,2,v);
    v:=dbms_random.value(1000,50);
    v_name:='paper'|| (i mod 3 + 1);
    v_order_detail_id:=v_order_detail_id+1;
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (v_order_detail_id,v_order_id,v_name,3,v);
    v:=dbms_random.value(9000,2000);
    v_name:='phone'|| (i mod 3 + 1);

    v_order_detail_id:=v_order_detail_id+1;
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (v_order_detail_id,v_order_id,v_name,1,v);
    --在触发器关闭的情况下，需要手工计算每个订单的应收金额：
    select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=v_order_id;
    if m is null then
     m:=0;
    end if;
    UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount WHERE ORDER_ID=v_order_id;
    IF I MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
  end loop;
end;
/
select count(*) from orders;
select count(*) from order_details;
```
结果如下：
```
PL/SQL 过程已成功完成。

  COUNT(*)
----------
     10000


  COUNT(*)
----------
     30000
```

在用system登陆，执行以下脚本：
```
ALTER USER cmq_user QUOTA UNLIMITED ON USERS;
ALTER USER cmq_user QUOTA UNLIMITED ON USERS02;
ALTER USER cmq_user QUOTA UNLIMITED ON USERS03;
```
执行结果：
```

User CMQ_USER已变更。


User CMQ_USER已变更。


User CMQ_USER已变更。
```
再回到我自己的用户下，执行以下代码：
```
declare
  dt date;
  m number(8,2);
  V_EMPLOYEE_ID NUMBER(6);
  v_order_id number(10);
  v_name varchar2(100);
  v_tel varchar2(100);
  v number(10,2);
  v_order_detail_id number;
begin
/*
system login:
ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS;
ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS02;
ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS03;
*/
  v_order_detail_id:=1;
  delete from order_details;
  delete from orders;
  for i in 1..10000
  loop
    if i mod 6 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2015
    elsif i mod 6 =1 then
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2016
    elsif i mod 6 =2 then
      dt:=to_date('2017-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2017
    elsif i mod 6 =3 then
      dt:=to_date('2018-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2018
    elsif i mod 6 =4 then
      dt:=to_date('2019-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2019
    else
      dt:=to_date('2020-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2020
    end if;
    V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
    --插入订单
    v_order_id:=i;
    v_name := 'aa'|| 'aa';
    v_name := 'zhang' || i;
    v_tel := '139888883' || i;
    insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
      values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
    --插入订单y一个订单包括3个产品
    v:=dbms_random.value(10000,4000);
    v_name:='computer'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (v_order_detail_id,v_order_id,v_name,2,v);
    v:=dbms_random.value(1000,50);
    v_name:='paper'|| (i mod 3 + 1);
    v_order_detail_id:=v_order_detail_id+1;
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (v_order_detail_id,v_order_id,v_name,3,v);
    v:=dbms_random.value(9000,2000);
    v_name:='phone'|| (i mod 3 + 1);

    v_order_detail_id:=v_order_detail_id+1;
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (v_order_detail_id,v_order_id,v_name,1,v);
    --在触发器关闭的情况下，需要手工计算每个订单的应收金额：
    select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=v_order_id;
    if m is null then
     m:=0;
    end if;
    UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount WHERE ORDER_ID=v_order_id;
    IF I MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
  end loop;
end;
/
select count(*) from orders;
select count(*) from order_details;
```
执行结果：
```
PL/SQL 过程已成功完成。

  COUNT(*)
----------
     10000


  COUNT(*)
----------
     30000
```
下面，再次进入system用户，执行以下代码1并查看执行计划：
```
set autotrace on

select * from cmq_user.orders where order_date
between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');
```
执行结果如下：
```
已启用自动跟踪
显示执行计划以及语句的统计信息。

  ORDER_ID CUSTOMER_NAME                            CUSTOMER_TEL                             ORDER_DAT EMPLOYEE_ID   DISCOUNT TRADE_RECEIVABLE
---------- ---------------------------------------- ---------------------------------------- --------- ----------- ---------- ----------------
         2 zhang2                                   1398888832                               04-3月 -17         112       1.44         15510.42
         8 zhang8                                   1398888838                               10-3月 -17         112      41.65         18710.25
        14 zhang14                                  13988888314                              16-3月 -17         112      25.36         21323.07
        20 zhang20                                  13988888320                              22-3月 -17         112      95.49         26354.69
        26 zhang26                                  13988888326                              28-3月 -17         112      35.44         23275.51
        32 zhang32                                  13988888332                              03-4月 -17         112      74.77         18892.61
        38 zhang38                                  13988888338                              09-4月 -17         112      92.66         13632.53
        44 zhang44                                  13988888344                              15-4月 -17         112       7.24         20701.56
        50 zhang50                                  13988888350                              21-4月 -17         112      40.29         18361.47
        56 zhang56                                  13988888356                              27-4月 -17         112       3.51         24670.15
        62 zhang62                                  13988888362                              04-3月 -17         112       98.8          25192.9
                                                                    ·
                                                                    ·
                                                                    ·
已选择 3,334 行。

Explain Plan
-----------------------------------------------------------

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
----------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 3103369360
 
---------------------------------------------------------------------------------------------------
| Id  | Operation                | Name   | Rows  | Bytes | Cost (%CPU)| Time     | Pstart| Pstop |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT         |        |     1 |   105 |   547   (0)| 00:00:01 |       |       |
|   1 |  PARTITION RANGE ITERATOR|        |     1 |   105 |   547   (0)| 00:00:01 |     3 |     4 |
|*  2 |   TABLE ACCESS FULL      | ORDERS |     1 |   105 |   547   (0)| 00:00:01 |     3 |     4 |
---------------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
----------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------
 
   2 - filter("ORDER_DATE"<=TO_DATE(' 2018-06-01 00:00:00', 'syyyy-mm-dd hh24:mi:ss'))
 
Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

Statistics
-----------------------------------------------------------
              33  Requests to/from client
              32  SQL*Net roundtrips to/from client
             604  bytes received via SQL*Net from client
          216695  bytes sent via SQL*Net to client
               4  calls to get snapshot scn: kcmgss
              44  calls to kcmgcs
             177  consistent gets
             177  consistent gets from cache
             177  consistent gets pin
             177  consistent gets pin (fastpath)
               1  enqueue releases
               1  enqueue requests
               2  execute count
         1449984  logical read bytes from cache
             135  no work - consistent read gets
              51  non-idle wait count
               2  opened cursors cumulative
               1  opened cursors current
               1  parse count (hard)
               2  parse count (total)
               1  recursive calls
               1  session cursor cache hits
             177  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
             135  table scan blocks gotten
            4378  table scan disk non-IMC rows gotten
            4378  table scan rows gotten
               2  table scans (short tables)
              33  user calls
```
然后，我们执行命令2：
```
已启用自动跟踪
显示执行计划以及语句的统计信息。

  ORDER_ID CUSTOMER_NAME                            PRODUCT_NAME                             PRODUCT_NUM PRODUCT_PRICE
---------- ---------------------------------------- ---------------------------------------- ----------- -------------
      7436 zhang7436                                computer3                                          2       7825.42
      7436 zhang7436                                paper3                                             3        259.08
      7436 zhang7436                                phone3                                             1       5025.42
      7442 zhang7442                                computer3                                          2       4879.22
      7442 zhang7442                                paper3                                             3        450.32
      7442 zhang7442                                phone3                                             1       5145.08
      7448 zhang7448                                computer3                                          2       5790.15
      7448 zhang7448                                paper3                                             3        392.14
      7448 zhang7448                                phone3                                             1       4978.18
      7454 zhang7454                                computer3                                          2       9943.31
      7454 zhang7454                                paper3                                             3        431.09
                                                        ·
                                                        ·
                                                        ·
脚本结果中当前只支持 5,000 行
已选择 5,000 行。

Explain Plan
-----------------------------------------------------------

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
----------------------------------------------------------------------------------------------------------------------------------
Plan hash value: 1648457282
 
----------------------------------------------------------------------------------------------------------
| Id  | Operation                | Name          | Rows  | Bytes | Cost (%CPU)| Time     | Pstart| Pstop |
----------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT         |               |     1 |   105 |   781   (0)| 00:00:01 |       |       |
|   1 |  PARTITION RANGE ITERATOR|               |     1 |   105 |   781   (0)| 00:00:01 |     3 |     4 |
|   2 |   NESTED LOOPS           |               |     1 |   105 |   781   (0)| 00:00:01 |       |       |
|*  3 |    TABLE ACCESS FULL     | ORDERS        |     1 |    44 |   547   (0)| 00:00:01 |     3 |     4 |
|*  4 |    TABLE ACCESS FULL     | ORDER_DETAILS |   296 | 18056 |   234   (0)| 00:00:01 |     3 |     4 |
----------------------------------------------------------------------------------------------------------

PLAN_TABLE_OUTPUT                                                                                                                                                                                                                                                                                           
----------------------------------------------------------------------------------------------------------------------------------
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   3 - filter("A"."ORDER_DATE"<=TO_DATE(' 2018-06-01 00:00:00', 'syyyy-mm-dd hh24:mi:ss'))
   4 - filter("A"."ORDER_ID"="B"."ORDER_ID")
 
Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

Statistics
-----------------------------------------------------------
               1  DB time
              36  Requests to/from client
              35  SQL*Net roundtrips to/from client
              10  buffer is not pinned count
             762  bytes received via SQL*Net from client
          190225  bytes sent via SQL*Net to client
              13  calls to get snapshot scn: kcmgss
             263  calls to kcmgcs
             474  consistent gets
             474  consistent gets from cache
             474  consistent gets pin
             474  consistent gets pin (fastpath)
               1  enqueue releases
               1  enqueue requests
               4  execute count
         3883008  logical read bytes from cache
             213  no work - consistent read gets
              45  non-idle wait count
               4  opened cursors cumulative
               1  opened cursors current
               1  parse count (hard)
               4  parse count (total)
               1  parse time elapsed
               7  recursive calls
               1  session cursor cache hits
             474  session logical reads
               1  sorts (memory)
            1178  sorts (rows)
             213  table scan blocks gotten
           11205  table scan disk non-IMC rows gotten
           11689  table scan rows gotten
              12  table scans (short tables)
              37  user calls

```
通过对比consistent gets，我们发现命令1花费了177，命令2花费了474，