# oralce
## 实验四
首先使用system用户给我的用户：cmq_user授予Create view权限：
```
grant Create View to  CMQ_user
```
结果如下：
```

Grant 成功。
```
然后，进入到我自己的用户，创建DEPARTMENTS、PRODUCTS、EMPLOYEES、ORDERS、ORDER_DETAILS与其索引。
首先先删除与之有关的表与索引，防止错误：
```
--删除表和序列
--删除表的同时会一起删除主外键、触发器、程序包。
declare
      num   number;
begin
      select count(1) into num from user_tables where TABLE_NAME = 'DEPARTMENTS';
      if   num=1   then
          execute immediate 'drop table DEPARTMENTS cascade constraints PURGE';
      end   if;

      select count(1) into num from user_tables where TABLE_NAME = 'EMPLOYEES';
      if   num=1   then
          execute immediate 'drop table EMPLOYEES cascade constraints PURGE';
      end   if;

      select count(1) into num from user_tables where TABLE_NAME = 'ORDER_ID_TEMP';
      if   num=1   then
          execute immediate 'drop table ORDER_ID_TEMP cascade constraints PURGE';
      end   if;

      select count(1) into num from user_tables where TABLE_NAME = 'ORDER_DETAILS';
      if   num=1   then
          execute immediate 'drop table ORDER_DETAILS cascade constraints PURGE';
      end   if;

      select count(1) into num from user_tables where TABLE_NAME = 'ORDERS';
      if   num=1   then
          execute immediate 'drop table ORDERS cascade constraints PURGE';
      end   if;

      select count(1) into num from user_tables where TABLE_NAME = 'PRODUCTS';
      if   num=1   then
          execute immediate 'drop table PRODUCTS cascade constraints PURGE';
      end   if;

      select count(1) into num from user_sequences where SEQUENCE_NAME = 'SEQ_ORDER_DETAILS_ID';
      if   num=1   then
          execute immediate 'drop  SEQUENCE SEQ_ORDER_DETAILS_ID';
      end   if;

      select count(1) into num from user_sequences where SEQUENCE_NAME = 'SEQ_ORDER_ID';
      if   num=1   then
          execute immediate 'drop  SEQUENCE SEQ_ORDER_ID';
      end   if;
      select count(1) into num from user_views where VIEW_NAME = 'VIEW_ORDER_DETAILS';
      if   num=1   then
          execute immediate 'drop VIEW VIEW_ORDER_DETAILS';
      end   if;

      SELECT count(object_name)  into num FROM user_objects_ae WHERE object_type = 'PACKAGE' and OBJECT_NAME='MYPACK';
      if   num=1   then
          execute immediate 'DROP PACKAGE MYPACK';
      end   if;
end;
```
结果如下：
```
PL/SQL 过程已成功完成。
```
* 接下来，我们就可以安心的创建表了,创建DEPARTMENTS表
```
--------------------------------------------------------
--  DDL for Table DEPARTMENTS
--------------------------------------------------------
CREATE TABLE DEPARTMENTS
(
  DEPARTMENT_ID NUMBER(6, 0) NOT NULL
, DEPARTMENT_NAME VARCHAR2(40 BYTE) NOT NULL
, CONSTRAINT DEPARTMENTS_PK PRIMARY KEY
  (
    DEPARTMENT_ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX DEPARTMENTS_PK ON DEPARTMENTS (DEPARTMENT_ID ASC)
      NOLOGGING
      TABLESPACE USERS
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        INITIAL 65536
        NEXT 1048576
        MINEXTENTS 1
        MAXEXTENTS UNLIMITED
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 1
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOCOMPRESS NO INMEMORY NOPARALLEL;
```
* 创建EMPLOYEES表，脚本如下:
```
--------------------------------------------------------
--  DDL for Table EMPLOYEES
--------------------------------------------------------
CREATE TABLE EMPLOYEES
(
  EMPLOYEE_ID NUMBER(6, 0) NOT NULL
, NAME VARCHAR2(40 BYTE) NOT NULL
, EMAIL VARCHAR2(40 BYTE)
, PHONE_NUMBER VARCHAR2(40 BYTE)
, HIRE_DATE DATE NOT NULL
, SALARY NUMBER(8, 2)
, MANAGER_ID NUMBER(6, 0)
, DEPARTMENT_ID NUMBER(6, 0)
, PHOTO BLOB
, CONSTRAINT EMPLOYEES_PK PRIMARY KEY
  (
    EMPLOYEE_ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX EMPLOYEES_PK ON EMPLOYEES (EMPLOYEE_ID ASC)
      NOLOGGING
      TABLESPACE USERS
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        INITIAL 65536
        NEXT 1048576
        MINEXTENTS 1
        MAXEXTENTS UNLIMITED
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 1
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NO INMEMORY
NOPARALLEL
LOB (PHOTO) STORE AS SYS_LOB0000092017C00009$$
(
  ENABLE STORAGE IN ROW
  CHUNK 8192
  NOCACHE
  NOLOGGING
  TABLESPACE USERS
  STORAGE
  (
    INITIAL 106496
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
);
```
* 创建索引EMPLOYEES_INDEX1_NAME：
```
CREATE INDEX EMPLOYEES_INDEX1_NAME ON EMPLOYEES (NAME ASC)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 2
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOPARALLEL;

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_FK1 FOREIGN KEY
(
  DEPARTMENT_ID
)
REFERENCES DEPARTMENTS
(
  DEPARTMENT_ID
)
ENABLE;

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_FK2 FOREIGN KEY
(
  MANAGER_ID
)
REFERENCES EMPLOYEES
(
  EMPLOYEE_ID
)
ON DELETE SET NULL ENABLE;

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_CHK1 CHECK
(SALARY>0)
ENABLE;

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_CHK2 CHECK
(EMPLOYEE_ID<>MANAGER_ID)
ENABLE;

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_EMPLOYEE_MANAGER_ID CHECK
(MANAGER_ID<>EMPLOYEE_ID)
ENABLE;

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_SALARY CHECK
(SALARY>0)
ENABLE;
```
* 创建PRODUCTS表
```
CREATE TABLE PRODUCTS
(
  PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
, PRODUCT_TYPE VARCHAR2(40 BYTE) NOT NULL
, CONSTRAINT PRODUCTS_PK PRIMARY KEY
  (
    PRODUCT_NAME
  )
  ENABLE
)
LOGGING
TABLESPACE "USERS"
PCTFREE 10
INITRANS 1
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS 2147483645
  BUFFER_POOL DEFAULT
);

ALTER TABLE PRODUCTS
ADD CONSTRAINT PRODUCTS_CHK1 CHECK
(PRODUCT_TYPE IN ('耗材', '手机', '电脑'))
ENABLE;
```
* 创建ORDER_ID_TEMP表
```
--------------------------------------------------------
--  DDL for Table ORDER_ID_TEMP
--------------------------------------------------------
CREATE GLOBAL TEMPORARY TABLE "ORDER_ID_TEMP"
   (	"ORDER_ID" NUMBER(10,0) NOT NULL ENABLE,
	 CONSTRAINT "ORDER_ID_TEMP_PK" PRIMARY KEY ("ORDER_ID") ENABLE
   ) ON COMMIT DELETE ROWS ;

   COMMENT ON TABLE "ORDER_ID_TEMP"  IS '用于触发器存储临时ORDER_ID';


--------------------------------------------------------
--  DDL for Table ORDERS
--------------------------------------------------------
CREATE TABLE ORDERS
(
  ORDER_ID NUMBER(10, 0) NOT NULL
, CUSTOMER_NAME VARCHAR2(40 BYTE) NOT NULL
, CUSTOMER_TEL VARCHAR2(40 BYTE) NOT NULL
, ORDER_DATE DATE NOT NULL
, EMPLOYEE_ID NUMBER(6, 0) NOT NULL
, DISCOUNT NUMBER(8, 2) DEFAULT 0
, TRADE_RECEIVABLE NUMBER(8, 2) DEFAULT 0
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
  PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
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
, PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USERS02
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
);

```
* 下面创建索引ORDERS_INDEX_DATE
```
--创建本地分区索引ORDERS_INDEX_DATE：
CREATE INDEX ORDERS_INDEX_DATE ON ORDERS (ORDER_DATE ASC)
LOCAL
(
  PARTITION PARTITION_BEFORE_2016
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
      INITIAL 8388608
      NEXT 1048576
      MINEXTENTS 1
      MAXEXTENTS UNLIMITED
      BUFFER_POOL DEFAULT
    )
    NOCOMPRESS
, PARTITION PARTITION_BEFORE_2017
    TABLESPACE USERS02
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
      INITIAL 8388608
      NEXT 1048576
      MINEXTENTS 1
      MAXEXTENTS UNLIMITED
      BUFFER_POOL DEFAULT
    )
    NOCOMPRESS
)
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOPARALLEL;

CREATE INDEX ORDERS_INDEX_CUSTOMER_NAME ON ORDERS (CUSTOMER_NAME ASC)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 2
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOPARALLEL;

CREATE UNIQUE INDEX ORDERS_PK ON ORDERS (ORDER_ID ASC)
GLOBAL PARTITION BY HASH (ORDER_ID)
(
  PARTITION INDEX_PARTITION1 TABLESPACE USERS
    NOCOMPRESS
, PARTITION INDEX_PARTITION2 TABLESPACE USERS02
    NOCOMPRESS
)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 2
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOPARALLEL;

ALTER TABLE ORDERS
ADD CONSTRAINT ORDERS_PK PRIMARY KEY
(
  ORDER_ID
)
USING INDEX ORDERS_PK
ENABLE;

ALTER TABLE ORDERS
ADD CONSTRAINT ORDERS_FK1 FOREIGN KEY
(
  EMPLOYEE_ID
)
REFERENCES EMPLOYEES
(
  EMPLOYEE_ID
)
ENABLE;
```
创建表ORDER_DETAILS
```
CREATE TABLE ORDER_DETAILS
(
  ID NUMBER(10, 0) NOT NULL
, ORDER_ID NUMBER(10, 0) NOT NULL
, PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
, PRODUCT_NUM NUMBER(8, 2) NOT NULL
, PRODUCT_PRICE NUMBER(8, 2) NOT NULL
, CONSTRAINT ORDER_DETAILS_FK1 FOREIGN KEY
  (
  ORDER_ID
  )
  REFERENCES ORDERS
  (
  ORDER_ID
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
PARTITION BY REFERENCE (ORDER_DETAILS_FK1)
(
  PARTITION PARTITION_BEFORE_2016
  NOLOGGING
  TABLESPACE USERS --必须指定表空间，否则会将分区存储在用户的默认表空间中
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
  NOCOMPRESS NO INMEMORY,
  PARTITION PARTITION_BEFORE_2017
  NOLOGGING
  TABLESPACE USERS02
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
)
;
```
创建索引
```
CREATE UNIQUE INDEX ORDER_DETAILS_PK ON ORDER_DETAILS (ID ASC)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 2
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOPARALLEL;

ALTER TABLE ORDER_DETAILS
ADD CONSTRAINT ORDER_DETAILS_PK PRIMARY KEY
(
  ID
)
USING INDEX ORDER_DETAILS_PK
ENABLE;
--这个索引是必须的，可以使整个订单的详单存放在一起
CREATE INDEX ORDER_DETAILS_ORDER_ID ON ORDER_DETAILS (ORDER_ID)
GLOBAL PARTITION BY HASH (ORDER_ID)
(
  PARTITION INDEX_PARTITION1 TABLESPACE USERS
    NOCOMPRESS
, PARTITION INDEX_PARTITION2 TABLESPACE USERS02
    NOCOMPRESS
);

ALTER TABLE ORDER_DETAILS
ADD CONSTRAINT ORDER_DETAILS_PRODUCT_NUM CHECK
(Product_Num>0)
ENABLE;
```
创建三个触发器
```
--创建3个触发器
--------------------------------------------------------
--  DDL for Trigger ORDERS_TRIG_ROW_LEVEL
--------------------------------------------------------
CREATE OR REPLACE EDITIONABLE TRIGGER "ORDERS_TRIG_ROW_LEVEL"
BEFORE INSERT OR UPDATE OF DISCOUNT ON "ORDERS"
FOR EACH ROW --行级触发器
declare
  m number(8,2);
BEGIN
  if inserting then
       :new.TRADE_RECEIVABLE := - :new.discount;
  else
      select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=:old.ORDER_ID;
      if m is null then
        m:=0;
      end if;
      :new.TRADE_RECEIVABLE := m - :new.discount;
  end if;
END;
```
插入需要的数据
```
--批量插入订单数据之前，禁用触发器
ALTER TRIGGER "ORDERS_TRIG_ROW_LEVEL" DISABLE;


--------------------------------------------------------
--  DDL for Trigger ORDER_DETAILS_ROW_TRIG
--------------------------------------------------------

CREATE OR REPLACE EDITIONABLE TRIGGER "ORDER_DETAILS_ROW_TRIG"
AFTER DELETE OR INSERT OR UPDATE  ON ORDER_DETAILS
FOR EACH ROW
BEGIN
  --DBMS_OUTPUT.PUT_LINE(:NEW.ORDER_ID);
  IF :NEW.ORDER_ID IS NOT NULL THEN
    MERGE INTO ORDER_ID_TEMP A
    USING (SELECT 1 FROM DUAL) B
    ON (A.ORDER_ID=:NEW.ORDER_ID)
    WHEN NOT MATCHED THEN
      INSERT (ORDER_ID) VALUES(:NEW.ORDER_ID);
  END IF;
  IF :OLD.ORDER_ID IS NOT NULL THEN
    MERGE INTO ORDER_ID_TEMP A
    USING (SELECT 1 FROM DUAL) B
    ON (A.ORDER_ID=:OLD.ORDER_ID)
    WHEN NOT MATCHED THEN
      INSERT (ORDER_ID) VALUES(:OLD.ORDER_ID);
  END IF;
END;
/
ALTER TRIGGER "ORDER_DETAILS_ROW_TRIG" DISABLE;
--------------------------------------------------------
--  DDL for Trigger ORDER_DETAILS_SNTNS_TRIG
--------------------------------------------------------

  CREATE OR REPLACE EDITIONABLE TRIGGER "ORDER_DETAILS_SNTNS_TRIG"
AFTER DELETE OR INSERT OR UPDATE ON ORDER_DETAILS
declare
  m number(8,2);
BEGIN
  FOR R IN (SELECT ORDER_ID FROM ORDER_ID_TEMP)
  LOOP
    --DBMS_OUTPUT.PUT_LINE(R.ORDER_ID);
    select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS
      where ORDER_ID=R.ORDER_ID;
    if m is null then
      m:=0;
    end if;
    UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount
      WHERE ORDER_ID=R.ORDER_ID;
  END LOOP;
  --delete from ORDER_ID_TEMP; --这句话很重要，否则可能一直不释放空间，后继插入会非常慢。
END;
/
ALTER TRIGGER "ORDER_DETAILS_SNTNS_TRIG" DISABLE;

--------------------------------------------------------
--  DDL for Sequence SEQ_ORDER_ID
--------------------------------------------------------
CREATE SEQUENCE  "SEQ_ORDER_ID"  MINVALUE 1 MAXVALUE 9999999999 INCREMENT BY 1 START WITH 1 CACHE 2000 ORDER  NOCYCLE  NOPARTITION ;
--------------------------------------------------------
--  DDL for Sequence SEQ_ORDER_DETAILS_ID
--------------------------------------------------------
CREATE SEQUENCE  "SEQ_ORDER_DETAILS_ID"  MINVALUE 1 MAXVALUE 9999999999 INCREMENT BY 1 START WITH 1 CACHE 2000 ORDER  NOCYCLE  NOPARTITION ;

--------------------------------------------------------
--  DDL for View VIEW_ORDER_DETAILS
--------------------------------------------------------
CREATE OR REPLACE FORCE EDITIONABLE VIEW "VIEW_ORDER_DETAILS" ("ID", "ORDER_ID", "CUSTOMER_NAME", "CUSTOMER_TEL", "ORDER_DATE", "PRODUCT_TYPE", "PRODUCT_NAME", "PRODUCT_NUM", "PRODUCT_PRICE") AS
  SELECT
  d.ID,
  o.ORDER_ID,
  o.CUSTOMER_NAME,o.CUSTOMER_TEL,o.ORDER_DATE,
  p.PRODUCT_TYPE,
  d.PRODUCT_NAME,
  d.PRODUCT_NUM,
  d.PRODUCT_PRICE
FROM ORDERS o,ORDER_DETAILS d,PRODUCTS p where d.ORDER_ID=o.ORDER_ID and d.PRODUCT_NAME=p.PRODUCT_NAME;
/

--插入DEPARTMENTS，EMPLOYEES数据
INSERT INTO DEPARTMENTS(DEPARTMENT_ID,DEPARTMENT_NAME) values (1,'总经办');
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (1,'李董事长',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,NULL,1);

INSERT INTO DEPARTMENTS(DEPARTMENT_ID,DEPARTMENT_NAME) values (11,'销售部1');
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (11,'张总',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,1,1);
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (111,'吴经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,11,11);
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (112,'白经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,11,11);

INSERT INTO DEPARTMENTS(DEPARTMENT_ID,DEPARTMENT_NAME) values (12,'销售部2');
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (12,'王总',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,1,1);
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (121,'赵经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,12,12);
INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
  VALUES (122,'刘经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,12,12);


insert into products (product_name,product_type) values ('computer1','电脑');
insert into products (product_name,product_type) values ('computer2','电脑');
insert into products (product_name,product_type) values ('computer3','电脑');

insert into products (product_name,product_type) values ('phone1','手机');
insert into products (product_name,product_type) values ('phone2','手机');
insert into products (product_name,product_type) values ('phone3','手机');

insert into products (product_name,product_type) values ('paper1','耗材');
insert into products (product_name,product_type) values ('paper2','耗材');
insert into products (product_name,product_type) values ('paper3','耗材');


--批量插入订单数据，注意ORDERS.TRADE_RECEIVABLE（订单应收款）的自动计算,注意插入数据的速度
--2千万条记录，插入的时间是：18100秒（约5小时）
declare
  dt date;
  m number(8,2);
  V_EMPLOYEE_ID NUMBER(6);
  v_order_id number(10);
  v_name varchar2(100);
  v_tel varchar2(100);
  v number(10,2);

begin
  for i in 1..10000
  loop
    if i mod 2 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60);
    else
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60);
    end if;
    V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
    --插入订单
    v_order_id:=SEQ_ORDER_ID.nextval; --应该将SEQ_ORDER_ID.nextval保存到变量中。
    v_name := 'aa'|| 'aa';
    v_name := 'zhang' || i;
    v_tel := '139888883' || i;
    insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
      values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
    --插入订单y一个订单包括3个产品
    v:=dbms_random.value(10000,4000);
    v_name:='computer'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,2,v);
    v:=dbms_random.value(1000,50);
    v_name:='paper'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,3,v);
    v:=dbms_random.value(9000,2000);
    v_name:='phone'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,1,v);
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
  --统计用户的所有表，所需时间很长：2千万行数据，需要1600秒，该语句可选
  --dbms_stats.gather_schema_stats(User,estimate_percent=>100,cascade=> TRUE); --estimate_percent采样行的百分比
end;
```
* 启用触发器
```
ALTER TRIGGER "ORDERS_TRIG_ROW_LEVEL" ENABLE;
ALTER TRIGGER "ORDER_DETAILS_SNTNS_TRIG" ENABLE;
ALTER TRIGGER "ORDER_DETAILS_ROW_TRIG" ENABLE;
```
* 添加一个动态增加一个PARTITION_BEFORE_2018分区
```
ALTER TABLE ORDERS
ADD PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'));

ALTER INDEX ORDERS_INDEX_DATE
MODIFY PARTITION PARTITION_BEFORE_2018
NOCOMPRESS;
```
* 下面开始测试，执行：
```
select * from ORDERS where  order_id=1;
```
查询结果
```

  ORDER_ID CUSTOMER_NAME                            CUSTOMER_TEL                             ORDER_DAT EMPLOYEE_ID   DISCOUNT TRADE_RECEIVABLE
---------- ---------------------------------------- ---------------------------------------- --------- ----------- ---------- ----------------
         1 zhang1                                   1398888831                               03-3月 -16         111      72.89         26155.64
```
执行：
```
select * from ORDER_DETAILS where  order_id=1;
```
结果：
```
        ID   ORDER_ID PRODUCT_NAME                             PRODUCT_NUM PRODUCT_PRICE
---------- ---------- ---------------------------------------- ----------- -------------
         1          1 computer2                                          2       7737.11
         2          1 paper2                                             3         697.3
         3          1 phone2                                             1       8662.41
```
执行：
```
        ID   ORDER_ID CUSTOMER_NAME                            CUSTOMER_TEL                             ORDER_DAT PRODUCT_TYPE                             PRODUCT_NAME                             PRODUCT_NUM PRODUCT_PRICE
---------- ---------- ---------------------------------------- ---------------------------------------- --------- ---------------------------------------- ---------------------------------------- ----------- -------------
         1          1 zhang1                                   1398888831                               03-3月 -16 电脑                                     computer2                                          2       7737.11
         2          1 zhang1                                   1398888831                               03-3月 -16 耗材                                     paper2                                             3         697.3
         3          1 zhang1                                   1398888831                               03-3月 -16 手机                                     phone2                                             1       8662.41
```
递归查询某个员工及其所有下属，子下属员工:
```
WITH A (EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID) AS
  (SELECT EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID
    FROM employees WHERE employee_ID = 11
    UNION ALL
  SELECT B.EMPLOYEE_ID,B.NAME,B.EMAIL,B.PHONE_NUMBER,B.HIRE_DATE,B.SALARY,B.MANAGER_ID,B.DEPARTMENT_ID
    FROM A, employees B WHERE A.EMPLOYEE_ID = B.MANAGER_ID)
SELECT * FROM A;
```
结果：
```
EMPLOYEE_ID NAME                                     EMAIL                                    PHONE_NUMBER                             HIRE_DATE     SALARY MANAGER_ID DEPARTMENT_ID
----------- ---------------------------------------- ---------------------------------------- ---------------------------------------- --------- ---------- ---------- -------------
         11 张总                                                                                                                       01-1月 -10      50000          1             1
        111 吴经理                                                                                                                     01-1月 -10      50000         11            11
        112 白经理                                                                                                                     01-1月 -10      50000         11            11
```
执行：
```
SELECT * FROM employees START WITH EMPLOYEE_ID = 11 CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID;
```
结果：
```
EMPLOYEE_ID NAME                                     EMAIL                                    PHONE_NUMBER                             HIRE_DATE     SALARY MANAGER_ID DEPARTMENT_ID PHOTO                                                                           
----------- ---------------------------------------- ---------------------------------------- ---------------------------------------- --------- ---------- ---------- ------------- --------------------------------------------------------------------------------
         11 张总                                                                                                                       01-1月 -10      50000          1             1                                                                                 
        111 吴经理                                                                                                                     01-1月 -10      50000         11            11                                                                                 
        112 白经理                                                                                                                     01-1月 -10      50000         11            11
```
查询分区表：
```
select TABLE_NAME,PARTITION_NAME,HIGH_VALUE,PARTITION_POSITION,TABLESPACE_NAME from user_tab_partitions
```
结果：
```
TABLE_NAME                                                                                                                       PARTITION_NAME                                                                                                                   HIGH_VALUE                                                                       PARTITION_POSITION TABLESPACE_NAME               
-------------------------------------------------------------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------- -------------------------------------------------------------------------------- ------------------ ------------------------------
ORDERS                                                                                                                           PARTITION_BEFORE_2016                                                                                                            TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIA                  1 USERS                         
ORDERS                                                                                                                           PARTITION_BEFORE_2017                                                                                                            TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIA                  2 USERS02                       
ORDERS                                                                                                                           PARTITION_BEFORE_2018                                                                                                            TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIA                  3 USERS                         
ORDER_DETAILS                                                                                                                    PARTITION_BEFORE_2016                                                                                                                                                                                                              1 USERS                         
ORDER_DETAILS                                                                                                                    PARTITION_BEFORE_2017                                                                                                                                                                                                              2 USERS02                       
ORDER_DETAILS                                                                                                                    PARTITION_BEFORE_2018                                                                                                                                                                                                              3 USERS                         

已选择 6 行。
```
查询分区索引情况：
```
INDEX_NAME                                                                                                                       COM PARTITION_NAME                                                                                                                   SUBPARTITION_COUNT HIGH_VALUE                                                                       HIGH_VALUE_LENGTH PARTITION_POSITION STATUS   TABLESPACE_NAME                  PCT_FREE  INI_TRANS  MAX_TRANS INITIAL_EXTENT NEXT_EXTENT MIN_EXTENT MAX_EXTENT   MAX_SIZE PCT_INCREASE  FREELISTS FREELIST_GROUPS LOGGING COMPRESSION       BLEVEL LEAF_BLOCKS DISTINCT_KEYS AVG_LEAF_BLOCKS_PER_KEY AVG_DATA_BLOCKS_PER_KEY CLUSTERING_FACTOR   NUM_ROWS SAMPLE_SIZE LAST_ANAL BUFFER_ FLASH_C CELL_FL USE PCT_DIRECT_ACCESS GLO DOMIDX PARAMETERS                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               INT SEG ORP
-------------------------------------------------------------------------------------------------------------------------------- --- -------------------------------------------------------------------------------------------------------------------------------- ------------------ -------------------------------------------------------------------------------- ----------------- ------------------ -------- ------------------------------ ---------- ---------- ---------- -------------- ----------- ---------- ---------- ---------- ------------ ---------- --------------- ------- ------------- ---------- ----------- ------------- ----------------------- ----------------------- ----------------- ---------- ----------- --------- ------- ------- ------- --- ----------------- --- ------ ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --- --- ---
ORDERS_INDEX_DATE                                                                                                                NO  PARTITION_BEFORE_2016                                                                                                                             0 TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIA                83                  1 USABLE   USERS                                  10          2        255        8388608     1048576          1 2147483645 2147483645                                         YES     DISABLED               0           0             0                       0                       0                 0          0             28-10月-19 DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  YES NO 
ORDERS_INDEX_DATE                                                                                                                NO  PARTITION_BEFORE_2017                                                                                                                             0 TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIA                83                  2 USABLE   USERS02                                10          2        255        8388608     1048576          1 2147483645 2147483645                                         YES     DISABLED               0           0             0                       0                       0                 0          0             28-10月-19 DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  YES NO 
ORDERS_INDEX_DATE                                                                                                                NO  PARTITION_BEFORE_2018                                                                                                                             0 TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIA                83                  3 USABLE   USERS                                  10          2        255                                                                                                     YES     DISABLED                                                                                                                                              DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  NO  NO 
ORDERS_PK                                                                                                                        NO  INDEX_PARTITION1                                                                                                                                  0                                                                                                  0                  1 USABLE   USERS                                  10          2        255          65536     1048576          1 2147483645 2147483645                                         NO      DISABLED               0           0             0                       0                       0                 0          0             28-10月-19 DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  YES NO 
ORDERS_PK                                                                                                                        NO  INDEX_PARTITION2                                                                                                                                  0                                                                                                  0                  2 USABLE   USERS02                                10          2        255          65536     1048576          1 2147483645 2147483645                                         NO      DISABLED               0           0             0                       0                       0                 0          0             28-10月-19 DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  YES NO 
ORDER_DETAILS_ORDER_ID                                                                                                           NO  INDEX_PARTITION1                                                                                                                                  0                                                                                                  0                  1 USABLE   USERS                                  10          2        255          65536     1048576          1 2147483645 2147483645                                         YES     DISABLED               0           0             0                       0                       0                 0          0             28-10月-19 DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  YES NO 
ORDER_DETAILS_ORDER_ID                                                                                                           NO  INDEX_PARTITION2                                                                                                                                  0                                                                                                  0                  2 USABLE   USERS02                                10          2        255          65536     1048576          1 2147483645 2147483645                                         YES     DISABLED               0           0             0                       0                       0                 0          0             28-10月-19 DEFAULT DEFAULT DEFAULT NO                    NO                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  NO  YES NO 

已选择 7 行。
```
查询一个分区中的数据
```
select count(*) from ORDERS partition(PARTITION_BEFORE_2016);
```
结果：
```
  COUNT(*)
----------
      5000
```
收集表的统计信息(统计用户的所有表):
```
exec dbms_stats.gather_schema_stats(User,estimate_percent=>100,cascade=> TRUE);
```
统计完成后，查询表的统计信息：
```
select table_name,tablespace_name,num_rows from user_tables where table_name='ORDERS';
select table_name,tablespace_name,num_rows from user_tables where table_name='ORDER_DETAILS';
select * from orders where order_id=1300;
select * from ORDER_DETAILS where order_id=1300;
select * from orders where customer_name='zhang133000';
select * from orders where order_date<to_date('2016-01-01','yyyy-mm-dd');
```
结果：
```
TABLE_NAME                                                                                                                       TABLESPACE_NAME                  NUM_ROWS
-------------------------------------------------------------------------------------------------------------------------------- ------------------------------ ----------
ORDERS                                                                                                                                                               10000


TABLE_NAME                                                                                                                       TABLESPACE_NAME                  NUM_ROWS
-------------------------------------------------------------------------------------------------------------------------------- ------------------------------ ----------
ORDER_DETAILS                                                                                                                                                        30000


  ORDER_ID CUSTOMER_NAME                            CUSTOMER_TEL                             ORDER_DAT EMPLOYEE_ID   DISCOUNT TRADE_RECEIVABLE
---------- ---------------------------------------- ---------------------------------------- --------- ----------- ---------- ----------------
      1300 zhang1300                                1398888831300                            11-4月 -15         121      46.67         22490.22


        ID   ORDER_ID PRODUCT_NAME                             PRODUCT_NUM PRODUCT_PRICE
---------- ---------- ---------------------------------------- ----------- -------------
      3898       1300 computer2                                          2       6998.97
      3899       1300 paper2                                             3        168.39
      3900       1300 phone2                                             1       8033.78

未选择任何行

  ORDER_ID CUSTOMER_NAME                            CUSTOMER_TEL                             ORDER_DAT EMPLOYEE_ID   DISCOUNT TRADE_RECEIVABLE
---------- ---------------------------------------- ---------------------------------------- --------- ----------- ---------- ----------------
      3768 zhang3768                                1398888833768                            19-4月 -15          11      14.73         26466.59
      3770 zhang3770                                1398888833770                            21-4月 -15         112      86.48         17961.09
      3772 zhang3772                                1398888833772                            23-4月 -15         121      93.81         29103.87
      3774 zhang3774                                1398888833774                            25-4月 -15          11       97.2         22650.21
      3776 zhang3776                                1398888833776                            27-4月 -15         112       48.9         20878.04
      3778 zhang3778                                1398888833778                            29-4月 -15         121      57.86         15303.47
      3780 zhang3780                                1398888833780                            02-3月 -15          11      40.17          25803.7
      3782 zhang3782                                1398888833782                            04-3月 -15         112      61.14         29298.12
      3784 zhang3784                                1398888833784                            06-3月 -15         121      51.49         19167.69
      3786 zhang3786                                1398888833786                            08-3月 -15          11       4.99          26159.8
      3788 zhang3788                                1398888833788                            10-3月 -15         112      66.68         14748.63
                                                                  ·
                                                                  ·
                                                                  ·
  ORDER_ID CUSTOMER_NAME                            CUSTOMER_TEL                             ORDER_DAT EMPLOYEE_ID   DISCOUNT TRADE_RECEIVABLE
---------- ---------------------------------------- ---------------------------------------- --------- ----------- ---------- ----------------
      8010 zhang8010                                1398888838010                            01-4月 -15          11      14.19         20433.97
      8012 zhang8012                                1398888838012                            03-4月 -15         112      56.98         26435.48
      8014 zhang8014                                1398888838014                            05-4月 -15         121      77.14         19807.77
      8016 zhang8016                                1398888838016                            07-4月 -15          11      11.17         15031.61
      8018 zhang8018                                1398888838018                            09-4月 -15         112      19.36         16135.27
      8020 zhang8020                                1398888838020                            11-4月 -15         121      75.32          16693.8

已选择 5,000 行。
```
## 执行查询语句
* 查询某个员工的信息，脚本如下：
```
select * from employees
```
结果：
```
EMPLOYEE_ID NAME                                     EMAIL                                    PHONE_NUMBER                             HIRE_DATE     SALARY MANAGER_ID DEPARTMENT_ID PHOTO                                                                           
----------- ---------------------------------------- ---------------------------------------- ---------------------------------------- --------- ---------- ---------- ------------- --------------------------------------------------------------------------------
          1 李董事长                                                                                                                   01-1月 -10      50000                        1                                                                                 
         11 张总                                                                                                                       01-1月 -10      50000          1             1                                                                                 
        111 吴经理                                                                                                                     01-1月 -10      50000         11            11                                                                                 
        112 白经理                                                                                                                     01-1月 -10      50000         11            11                                                                                 
         12 王总                                                                                                                       01-1月 -10      50000          1             1                                                                                 
        121 赵经理                                                                                                                     01-1月 -10      50000         12            12                                                                                 
        122 刘经理                                                                                                                     01-1月 -10      50000         12            12                                                                                 

已选择 7 行。
```
* 递归查询某个员工及其所有下属，子下属员工:
```
WITH A (EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID) AS
  (SELECT EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID
    FROM employees WHERE employee_ID = 11
    UNION ALL
  SELECT B.EMPLOYEE_ID,B.NAME,B.EMAIL,B.PHONE_NUMBER,B.HIRE_DATE,B.SALARY,B.MANAGER_ID,B.DEPARTMENT_ID
    FROM A, employees B WHERE A.EMPLOYEE_ID = B.MANAGER_ID)
SELECT * FROM A;
```
结果：
```
EMPLOYEE_ID NAME                                     EMAIL                                    PHONE_NUMBER                             HIRE_DATE     SALARY MANAGER_ID DEPARTMENT_ID PHOTO                                                                           
----------- ---------------------------------------- ---------------------------------------- ---------------------------------------- --------- ---------- ---------- ------------- --------------------------------------------------------------------------------
         11 张总                                                                                                                       01-1月 -10      50000          1             1                                                                                 
        111 吴经理                                                                                                                     01-1月 -10      50000         11            11                                                                                 
        112 白经理
```
* 查询订单表，并且包括订单的订单应收货款: Trade_Receivable= sum(订单详单表.ProductNum*订单详单表.ProductPrice)- Discount:
```
--计算由行触发器完成
select order_id,trade_receivable from ORDERS
```
结果:
```
ORDER_ID TRADE_RECEIVABLE
---------- ----------------
      3768         26466.59
      3770         17961.09
      3772         29103.87
      3774         22650.21
      3776         20878.04
      3778         15303.47
      3780          25803.7
      3782         29298.12
      3784         19167.69
      3786          26159.8
      3788         14748.63
            ·
            ·
            ·
```
* 查询订单详表，要求显示订单的客户名称和客户电话，产品类型用汉字描述:
```
select a.order_id as "订单编号",a.customer_name as "顾客名字",a.customer_tel as "联系电话",b.product_type as "产品类型"
from ORDERS a,
(select order_details.order_id,products.product_type from products,order_details 
where products.product_name = order_details.product_name) b
where a.order_id =b.order_id 
```
结果：
```
 订单编号 顾客名字                                     联系电话                                     产品类型                                    
---------- ---------------------------------------- ---------------------------------------- ----------------------------------------
      2344 zhang2344                                1398888832344                            手机                                    
      2346 zhang2346                                1398888832346                            电脑                                    
      2346 zhang2346                                1398888832346                            耗材                                    
      2346 zhang2346                                1398888832346                            手机                                    
      2348 zhang2348                                1398888832348                            电脑                                    
      2348 zhang2348                                1398888832348                            耗材                                    
      2348 zhang2348                                1398888832348                            手机                                    
      2350 zhang2350                                1398888832350                            电脑                                    
      2350 zhang2350                                1398888832350                            耗材                                    
      2350 zhang2350                                1398888832350                            手机                                    
      2352 zhang2352                                1398888832352                            电脑
                                                          ·
                                                          ·
                                                          ·
```
* 查询出所有空订单，即没有订单详单的订单,首先插入空订单信息:
```
ALTER TRIGGER "ORDERS_TRIG_ROW_LEVEL" DISABLE;
declare
  dt date;
  m number(8,2);
  V_EMPLOYEE_ID NUMBER(6);
  v_order_id number(10);
  v_name varchar2(100);
  v_tel varchar2(100);
  v number(10,2);

begin
  for i in 1000..1010
  loop
    if i mod 2 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60);
    else
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60);
    end if;
    V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
    --插入订单
    v_order_id:=SEQ_ORDER_ID.nextval; --应该将SEQ_ORDER_ID.nextval保存到变量中。
    v_name := 'aa'|| 'aa';
    v_name := '陈泯全的空订单' || i;
    v_tel := '139888883' || i;
    insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
      values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
    
  end loop;
  --统计用户的所有表，所需时间很长：2千万行数据，需要1600秒，该语句可选
  --dbms_stats.gather_schema_stats(User,estimate_percent=>100,cascade=> TRUE); --estimate_percent采样行的百分比
end;
```
然后查询空订单：
```
select order_id,customer_name from orders where order_id not in (select order_id from order_details)
```
结果：
```
ORDER_ID CUSTOMER_NAME                           
---------- ----------------------------------------
     10007 陈泯全的空订单1006                      
     10001 陈泯全的空订单1000                      
     10011 陈泯全的空订单1010                      
     10008 陈泯全的空订单1007                      
     10004 陈泯全的空订单1003                      
     10010 陈泯全的空订单1009                      
     10009 陈泯全的空订单1008                      
     10005 陈泯全的空订单1004                      
     10003 陈泯全的空订单1002                      
     10002 陈泯全的空订单1001                      
     10006 陈泯全的空订单1005                      

已选择 11 行。
```
* 查询部门表，同时显示部门的负责人姓名。
```

```
结果
```

```
* 查询部门表，统计每个部门的销售总金额:
```
select DEPARTMENT_NAME as "部门名字",总销售 from departments a,
(select a.DEPARTMENT_ID,sum(b.sumNum) as "总销售" from employees a,
(select EMPLOYEE_ID,sum(TRADE_RECEIVABLE) as sumNum  from orders group by EMPLOYEE_ID) b
GROUP by DEPARTMENT_ID) b
where a.DEPARTMENT_ID =b.DEPARTMENT_ID
```
结果：
```
部门名字                                            总销售
---------------------------------------- ----------
总经办                                    630681697
销售部1                                   420454465
销售部2                                   420454465
```