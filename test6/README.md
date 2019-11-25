# 任务1：表，表空间设计合理，数据合理



1. 新键一个PDB：weibo

```sql
CREATE PLUGGABLE DATABASE weibo ADMIN USER admin IDENTIFIED BY 123 FILE_NAME_CONVERT=('home/oracle/app/oracle/oradata/orcl/pdbseed','home/oracle/app/oracle/oradata/orcl/weobo');
```

1. 划分表空间，划分出三个表空间

    1. weibo_1.dbf大小500M 每次100M 无限大小 （用于存放大量数据：4W）

    1. weibo_2.dbf大小 500M 每次 100M 无限大小 （用于存放大量数据4W）

    1. weibo_3.dbf 大小 100M 每次 50M 无限大小   （用于存放少量数据 1W）

```text
create TABLESPACE users02 DATAFILE DATABASE '/home/oracle/app/oracle/oradata/orcl/weobo/weibo_1.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED, '/home/oracle/app/oracle/oradata/orcl/weobo/weibo_2.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED EXTENT MANAGEMENT 
'/home/oracle/app/oracle/oradata/orcl/weobo/weibo_3.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

** 结果：**

![](https://tcs-ga.teambition.net/storage/111n88473458a2466406c04af50174d6be42?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTU3NTExMjQxNywiaWF0IjoxNTc0NTA3NjE3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMW44ODQ3MzQ1OGEyNDY2NDA2YzA0YWY1MDE3NGQ2YmU0MiJ9.ZMWPJbuZAWkBvmmwsQV7GZhEo9YNblEyqB4VCseBE2Y&download=image.png "")

![](https://tcs-ga.teambition.net/storage/111n34b01a65bcec58b15bcda2022d59132e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTU3NTExMjQxNywiaWF0IjoxNTc0NTA3NjE3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMW4zNGIwMWE2NWJjZWM1OGIxNWJjZGEyMDIyZDU5MTMyZSJ9.7p6PUwBePb-_MeLs3uG9lkVqPDhTeEuP8jY0CkkwKC4&download=image.png "")

1. 数据库设计

*管理员表、收藏表、评论表、喜欢表、位置表、mention表、关系表、回复表、用户表、微博表*

|  表名             |   功能说明     |
| --------------- | ---------- |
|   admin         |   管理员表     |
|   collect       |   用于存储地层参数 |
|   comment       |   收藏表      |
|   likes         |   喜欢表      |
|   location      |   位置表      |
| mention mention |  mention表  |
|  relation       |  关系表       |
|  reply          |  回复表       |
|  user           |  用户表       |
|  weibo          |  微博表       |

| 表名   | admin |          |             |      |
| ---- | ----- | -------- | ----------- | ---- |
| 字段编号 | 物理字段  | 逻辑字段     | 字段类型        | 是否主键 |
| 1    | 管理员编号 | admin_id | Int         | Y    |
| 2    | 用户名   | username | varchar(50) |      |
| 3    | 密码    | passwrod | varchar(50) |      |

```text
  CREATE TABLE "SYSTEM"."ADMIN" 
   (	"ADMIN_ID" NUMBER(*,0) NOT NULL ENABLE, 
	"COLUMN1" VARCHAR2(50 BYTE), 
	"PASSWORD" VARCHAR2(50 BYTE)
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | collect |              |          |      |
| ---- | ------- | ------------ | -------- | ---- |
| 字段编号 | 物理字段    | 逻辑字段         | 字段类型     | 是否主键 |
| 1    | 收藏ID    | collect_id   | Int      | Y    |
| 2    | 微博编号    | weibo_id     | Int      |      |
| 3    | 收藏时间    | collcet_time | datatime |      |

```text
  CREATE TABLE "SYSTEM"."COLLECT" 
   (	"COLLECT_ID" NUMBER(*,0) NOT NULL ENABLE, 
	"WEIBO_ID" NUMBER(*,0), 
	"COLLECT_TIME" DATE
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | comment_weibo |                 |             |      |      |
| ---- | ------------- | --------------- | ----------- | ---- | ---- |
| 字段编号 | 物理字段          | 逻辑字段            | 字段类型        | 是否主键 | 是否外键 |
| 1    | 评论表id         | comment_id      | Int         | Y    |      |
| 2    | 用户id          | user_id         | Int         |      | Y    |
| 3    | 微博id          | weibo_id        | Int         |      |      |
| 4    | 评论时间          | commment_time   | datatime    |      |      |
| 5    | 评论内容          | comment_content | varchar(50) |      |      |

```text
  CREATE TABLE "SYSTEM"."COMMENT_WEIBO" 
   (	"COMMENT_ID" NUMBER NOT NULL ENABLE, 
	"USER_ID" NUMBER, 
	"COLUMN1" NUMBER, 
	"COMMENT_TIME" DATE, 
	"COLUMN2" VARCHAR2(50 BYTE)
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | likes |            |          |      |      |
| ---- | ----- | ---------- | -------- | ---- | ---- |
| 字段编号 | 物理字段  | 逻辑字段       | 字段类型     | 是否主键 | 是否外键 |
| 1    | 喜欢id  | like_id    | Int      | Y    |      |
| 2    | 用户id  | user_id    | Int      |      | Y    |
| 3    | 微博id  | weibo_id   | Int      |      |      |
| 4    | 喜欢时间  | likes_time | datatime |      |      |

```text
  CREATE TABLE "SYSTEM"."LIKES" 
   (	"LIKOES_ID" NUMBER NOT NULL ENABLE, 
	"USER_ID" NUMBER, 
	"WEIBO_ID" NUMBER, 
	"LIKE_TIME" DATE
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | location |         |             |      |      |
| ---- | -------- | ------- | ----------- | ---- | ---- |
| 字段编号 | 物理字段     | 逻辑字段    | 字段类型        | 是否主键 | 是否外键 |
| 1    | id       | like_id | Int         | Y    |      |
| 2    | 名字       | name    | varchar(50) |      | Y    |

```text
  CREATE TABLE "SYSTEM"."LOCATION" 
   (	"ID" NUMBER NOT NULL ENABLE, 
	"NAME" VARCHAR2(50 BYTE)
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | mention |              |      |      |      |
| ---- | ------- | ------------ | ---- | ---- | ---- |
| 字段编号 | 物理字段    | 逻辑字段         | 字段类型 | 是否主键 | 是否外键 |
| 1    | 提到id    | mention_id   | Int  | Y    |      |
| 2    | 用户id    | user_id      | Int  |      | Y    |
| 3    | 报告      | reportCount  | Int  |      |      |
| 4    | 评论条数    | commentCount | Int  |      |      |
| 5    | 回复条数    | replayCount  | Int  |      |      |
| 6    | 喜欢条数    | likeCount    | Int  |      |      |
| 7    | 粉丝数     | fansCount    | Int  |      |      |

```text
  CREATE TABLE "SYSTEM"."MENTION" 
   (	"MENTION_ID" NUMBER NOT NULL ENABLE, 
	"COLUMN1" NUMBER, 
	"REPORTCOUNT" NUMBER, 
	"COMMENTCOUNT" NUMBER, 
	"A" NUMBER, 
	"LIKECOUNT" NUMBER, 
	"FANSCOUNT" VARCHAR2(20 BYTE)
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | relation |             |      |      |      |
| ---- | -------- | ----------- | ---- | ---- | ---- |
| 字段编号 | 物理字段     | 逻辑字段        | 字段类型 | 是否主键 | 是否外键 |
| 1    | 关系id     | relation_id | Int  | Y    |      |
| 2    | 用户id     | user_id     | Int  |      | Y    |
| 3    | 关注id     | followid    | Int  |      |      |
| 4    | 状态       | state       | Int  |      |      |

```text
  CREATE TABLE "SYSTEM"."RELATION" 
   (	"RELATION_ID" NUMBER NOT NULL ENABLE, 
	"USER_ID" NUMBER, 
	"FOLLOWID" NUMBER, 
	"STATE" NUMBER
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | replay |                |             |      |      |
| ---- | ------ | -------------- | ----------- | ---- | ---- |
| 字段编号 | 物理字段   | 逻辑字段           | 字段类型        | 是否主键 | 是否外键 |
| 1    | 回复id   | replay_id      | Int         | Y    |      |
| 2    | 评论id   | comment_id     | Int         |      | Y    |
| 3    | 表单id   | from_id        | Int         |      |      |
| 4    | 评论对象   | ro_id          | Int         |      |      |
| 5    | 评论时间   | replay_content | varchar(50) |      |      |
| 6    | 时间     | time           | data        |      |      |

```text
  CREATE TABLE "SYSTEM"."REPLAY" 
   (	"RELAY_ID" NUMBER NOT NULL ENABLE, 
	"COMMENT_ID" NUMBER, 
	"FROM_ID" NUMBER, 
	"TO_ID" VARCHAR2(50 BYTE), 
	"REPLAY_CONTENT" VARCHAR2(20 BYTE), 
	"TIME" DATE
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

| 表名   | user_weibo |          |             |      |      |
| ---- | ---------- | -------- | ----------- | ---- | ---- |
| 字段编号 | 物理字段       | 逻辑字段     | 字段类型        | 是否主键 | 是否外键 |
| 1    | 用户id       | user_id  | Int         | Y    |      |
| 2    | 用户名        | username | varchar(50) |      | Y    |
| 3    | 密码         | password | varchar(50) |      |      |
| 4    | 昵称         | nickname | varchar(50) |      |      |
| 5    | 头像         | face     | varchar(50) |      |      |
| 6    | 性别         | sex      | varchar(50) |      |      |
| 7    | 生日         | bir      | data        |      |      |
| 8    | 省份         | province | varchar(50) |      |      |
| 9    | 城市         | city     | varchar(50) |      |      |

```text

  CREATE TABLE "SYSTEM"."USER_WIEBO" 
   (	"USER_ID" NUMBER NOT NULL ENABLE, 
	"USERNAME" VARCHAR2(50 BYTE), 
	"PASSWORD" VARCHAR2(50 BYTE), 
	"NICKNAME" VARCHAR2(50 BYTE), 
	"FACE" VARCHAR2(50 BYTE), 
	"SEX" VARCHAR2(50 BYTE), 
	"BIR" DATE, 
	"PROVINCE" VARCHAR2(50 BYTE), 
	"CITY" VARCHAR2(50 BYTE)
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;

```

| 表名   | weibo     |          |             |      |      |
| ---- | --------- | -------- | ----------- | ---- | ---- |
| 字段编号 | 物理字段      | 逻辑字段     | 字段类型        | 是否主键 | 是否外键 |
| 1    | 微博id      | weibo_id | Int         | Y    |      |
| 2    | 用户id      | suer_id  | Int         |      | Y    |
| 3    | post_time | 发送时间     | varchar(50) |      |      |
| 4    | content   | 内容       | varchar(50) |      |      |
| 5    | pic1      | 图片       | varchar(50) |      |      |
| 6    | pic2      | 图片       | varchar(50) |      |      |
| 7    | pic3      | 图片       | varchar(50) |      |      |
| 8    | pic4      | 图片       | varchar(50) |      |      |
| 9    | original  | int      | varchar(50) |      |      |
| 10   | repost_id | int      | varchar(50) |      |      |

```text

  
  CREATE TABLE "SYSTEM"."WEIBO" 
   (	"WEIBO_ID" NUMBER NOT NULL ENABLE, 
	"USERNAME" VARCHAR2(20 BYTE), 
	"POST_TIME" DATE, 
	"CONTENT" VARCHAR2(50 BYTE), 
	"PIC1" VARCHAR2(20 BYTE), 
	"PIC2" VARCHAR2(20 BYTE), 
	"PIC3" VARCHAR2(20 BYTE), 
	"PIC4" VARCHAR2(20 BYTE), 
	"ORIGINAL" NUMBER, 
	"REPOST_ID" NUMBER
   ) PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
 NOCOMPRESS LOGGING
  STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
  PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
  BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
  TABLESPACE "SYSTEM" ;
```

- 写入admin数据：1W条：

```text
declare
  v_order_id number(10);
  v_name varchar2(100);
begin
    delete from ADMIN;
    for i in 1..10000
    loop
        v_order_id:=i;
        v_name := 'zhang' || i;
        insert /*+append*/ into ADMIN (ADMIN.ADMIN_ID,ADMIN.USERNAME,ADMIN.PASSWORD) values (v_order_id,v_name,'123456');
    end loop;
end;
```

- 写入user_weibo数据：2W条：

```text
declare
  v_id number(10);
  v_name varchar2(100);
  v_password varchar2(50);
  v_nicknane varchar(100);
  v_face varchar(100);
  v_sex varchar(2);
  v_bir varchar(100);
  v_province varchar(100);
  v_city varchar(100);
begin
    delete from ADMIN;
    for i in 1..10000
    loop
        v_id:=i;
        v_name := 'zhang' || i;
        v_nicknane:= 'zhang' || i;
        v_face:='asdfafasdff';
        v_sex:=1;
        v_bir:='1997.11.1';
        v_province:='sichuan';
        v_city :='chengDu';
        insert /*+append*/ into user_weibo (user_weibo.USER_ID,user_weibo.username,user_weibo.password,
        user_weibo.nickname,user_weibo.face,user_weibo.sex,user_weibo.province,user_weibo.CITY) 
        values (v_id,v_name,'123456',v_nicknane,v_face,v_sex,v_province,v_city);
    end loop;
end;
```

- 导入weibo数据：4W条

```text
declare
  v_id number(10);
  v_user_id varchar2(100);
  v_content varchar2(50);
begin
    delete from ADMIN;
    for i in 1..40000
    loop
        v_id:=i;
        v_user_id := 1;
        v_content:= 'testtetsetesteststs';

        insert /*+append*/ into weibo (weibo.WEIBO_ID,weibo.USERNAME,weibo.CONTENT) 
        values (v_id,v_user_id,v_content);
    end loop;
end;
```

# 任务2：权限及用户分配方案设计

1. 创建1个角色

    - 管理员角色：CON_RES,并赋予RESOURCE,CONNECT角色

```sql
CREATE ROLE con_res;
grant connect,resource to con_res;
```

1. 创建4个用户：

    1. admin：管理员用户并将所有权限赋予它

```sql
GRANT "DBA" TO "ADMIN" WITH ADMIN OPTION;
GRANT "C##CON_RES" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_MONITOR" TO "ADMIN" WITH ADMIN OPTION;
GRANT "CTXAPP" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_AUDIT_CLEANUP" TO "ADMIN" WITH ADMIN OPTION;
GRANT "SPATIAL_CSW_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "APEX_GRANTS_FOR_NEW_USERS_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "WFS_USR_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "EM_EXPRESS_ALL" TO "ADMIN" WITH ADMIN OPTION;
GRANT "WM_ADMIN_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "OLAP_USER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "OLAP_XS_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_SECANALYST" TO "ADMIN" WITH ADMIN OPTION;
GRANT "CSW_USR_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XS_CACHE_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "GDS_CATALOG_SELECT" TO "ADMIN" WITH ADMIN OPTION;
GRANT "SCHEDULER_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "PROVISIONER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "AUDIT_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XDB_WEBSERVICES_OVER_HTTP" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_REALM_RESOURCE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "AQ_ADMINISTRATOR_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DELETE_CATALOG_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "CON_RES" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XDB_WEBSERVICES" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_PUBLIC" TO "ADMIN" WITH ADMIN OPTION;
GRANT "LBAC_DBA" TO "ADMIN" WITH ADMIN OPTION;
GRANT "OPTIMIZER_PROCESSING_RATE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "RECOVERY_CATALOG_USER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_DATAPUMP_NETWORK_LINK" TO "ADMIN" WITH ADMIN OPTION;
GRANT "GSMUSER_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "GATHER_SYSTEM_STATISTICS" TO "ADMIN" WITH ADMIN OPTION;
GRANT "LOGSTDBY_ADMINISTRATOR" TO "ADMIN" WITH ADMIN OPTION;
GRANT "GSM_POOLADMIN_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "HS_ADMIN_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XS_SESSION_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_GOLDENGATE_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "IMP_FULL_DATABASE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_XSTREAM_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_PATCH_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DATAPUMP_EXP_FULL_DATABASE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "EJBCLIENT" TO "ADMIN" WITH ADMIN OPTION;
GRANT "HS_ADMIN_EXECUTE_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JMXSERVER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "OLAP_DBA" TO "ADMIN" WITH ADMIN OPTION;
GRANT "ADM_PARALLEL_EXECUTE_TASK" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JAVAIDPRIV" TO "ADMIN" WITH ADMIN OPTION;
GRANT "SELECT_CATALOG_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JAVADEBUGPRIV" TO "ADMIN" WITH ADMIN OPTION;
GRANT "CONNECT" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DATAPUMP_IMP_FULL_DATABASE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "OEM_MONITOR" TO "ADMIN" WITH ADMIN OPTION;
GRANT "APEX_ADMINISTRATOR_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "GSMADMIN_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "AQ_USER_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JAVAUSERPRIV" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XDB_SET_INVOKER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "RECOVERY_CATALOG_OWNER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JAVA_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DBFS_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "PDB_DBA" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_GOLDENGATE_REDO_ACCESS" TO "ADMIN" WITH ADMIN OPTION;
GRANT "CDB_DBA" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JAVASYSPRIV" TO "ADMIN" WITH ADMIN OPTION;
GRANT "HS_ADMIN_SELECT_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "AUDIT_VIEWER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "RESOURCE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_OWNER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XDB_WEBSERVICES_WITH_PUBLIC" TO "ADMIN" WITH ADMIN OPTION;
GRANT "EXECUTE_CATALOG_ROLE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_ACCTMGR" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_REALM_OWNER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "EXP_FULL_DATABASE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DV_STREAMS_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "JAVA_DEPLOY" TO "ADMIN" WITH ADMIN OPTION;
GRANT "SPATIAL_WFS_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XS_NAMESPACE_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "DEV" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XS_RESOURCE" TO "ADMIN" WITH ADMIN OPTION;
GRANT "ORDADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "AUTHENTICATEDUSER" TO "ADMIN" WITH ADMIN OPTION;
GRANT "CAPTURE_ADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "OEM_ADVISOR" TO "ADMIN" WITH ADMIN OPTION;
GRANT "XDBADMIN" TO "ADMIN" WITH ADMIN OPTION;
GRANT "EM_EXPRESS_BASIC" TO "ADMIN" WITH ADMIN OPTION;
```

    1. zhangdev：开发用户1，只赋予weibo,user_weibo表的所有权限给这个开发者

```sql
CREATE USER zhangdev IDENTIFIED BY "123"
grant all on weibo to zhangdev;
grant all on user_weibo to zhangdev;
```

    1. chengdev：开发用户2，赋予开发用户1 的权限，同时开放所有的index,select,debug,update,delete对象给这个用户

```sql
CREATE USER chengdev IDENTIFIED BY "123"  
grant all on user_weibo  to chengdev;
grant all on weibo  to chengdev;
grant  index,select,debug,update,delete  on admin to zhangdev;
grant  index,select,debug,update,delete  on collect to zhangdev;
grant  index,select,debug,update,delete  on comment_weibo to zhangdev;
grant  index,select,debug,update,delete  on likes to zhangdev;
grant  index,select,debug,update,delete  on location to zhangdev;
grant  index,select,debug,update,delete  on mention  to zhangdev;
grant  index,select,debug,update,delete  on relation to zhangdev;
grant  index,select,debug,update,delete  on replay  to zhangdev;
```

    1. wangtest：测试用户

```sql
CREATE USER wangtest IDENTIFIED BY "123"  
grant  select,debug on admin to wangtest;
grant  select,debug,delete  on collect to wangtest;
grant  select,debug  on comment_weibo to wangtest;
grant  select,debug on likes to wangtest;
grant  select,debug  on location to wangtest;
grant  select,debug on mention  to wangtest;
grant  select,debug on relation to wangtest;
grant  select,debug on replay  to wangtest;
grant  select,debug on user_weibo  to wangtest;
grant  select,debug on seibo  to wangtest;
```

# 任务3：存储过程和函数

1. 函数1

```sql
create or replace FUNCTION Get_weibo(v_id  NUMBER) RETURN NUMBER
AS
    N NUMBER;
    BEGIN
        SELECT COUNT(*) INTO N from weibo where weibo.weibo_id=v_id;
        RETURN N;
    END;
```

1. 存储过程

```sql
create or replace PROCEDURE GET_user(v_id NUMBER, p_cur out sys_refcursor) 
AS
BEGIN
   open p_cur for SELECT *FROM user_weibo where user_weibo.user_id=v_id;
END;
```

任务4：备份方案

1、数据逻辑备份

```text
expdp study/123@pdborcl directory=expdir dumpfile=study.dmp
```

结果：

```text
Export: Release 12.1.0.2.0 - Production on 星期一 11月 25 10:42:17 2019

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
ORA-39001: 参数值无效
ORA-39000: 转储文件说明错误
ORA-31641: 无法创建转储文件 "/home/oracle/expdir/study.dmp"
ORA-27038: 所创建的文件已存在
Additional information: 1


[oracle@oracle-pc Desktop]$ expdp study/123@pdborcl directory=expdir dumpfile=study.dmp

Export: Release 12.1.0.2.0 - Production on 星期一 11月 25 10:43:40 2019

Copyright (c) 1982, 2014, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
启动 "STUDY"."SYS_EXPORT_SCHEMA_01":  study/********@pdborcl directory=expdir dumpfile=study.dmp 
正在使用 BLOCKS 方法进行估计...
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE_DATA
使用 BLOCKS 方法的总估计: 33.31 MB
处理对象类型 SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
处理对象类型 SCHEMA_EXPORT/SEQUENCE/SEQUENCE
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE
处理对象类型 SCHEMA_EXPORT/TABLE/COMMENT
处理对象类型 SCHEMA_EXPORT/PACKAGE/PACKAGE_SPEC
处理对象类型 SCHEMA_EXPORT/PROCEDURE/PROCEDURE
处理对象类型 SCHEMA_EXPORT/PACKAGE/COMPILE_PACKAGE/PACKAGE_SPEC/ALTER_PACKAGE_SPEC
处理对象类型 SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
处理对象类型 SCHEMA_EXPORT/VIEW/VIEW
处理对象类型 SCHEMA_EXPORT/PACKAGE/PACKAGE_BODY
处理对象类型 SCHEMA_EXPORT/TABLE/INDEX/INDEX
处理对象类型 SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
处理对象类型 SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
处理对象类型 SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
处理对象类型 SCHEMA_EXPORT/TABLE/TRIGGER
处理对象类型 SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
处理对象类型 SCHEMA_EXPORT/STATISTICS/MARKER
处理对象类型 SCHEMA_EXPORT/POST_SCHEMA/PROCOBJ
. . 导出了 "STUDY"."ORDERS":"PARTITION_BEFORE_2016"    268.4 KB    5000 行
. . 导出了 "STUDY"."ORDERS":"PARTITION_BEFORE_2017"    268.5 KB    5000 行
. . 导出了 "STUDY"."EMPLOYEES"                         70.67 KB       7 行
. . 导出了 "STUDY"."DEPARTMENTS"                       5.593 KB       3 行
. . 导出了 "STUDY"."PRODUCTS"                          6.296 KB      15 行
. . 导出了 "STUDY"."ORDERS":"PARTITION_BEFORE_2018"        0 KB       0 行
. . 导出了 "STUDY"."ORDER_DETAILS":"PARTITION_BEFORE_2016"  425.7 KB   15000 行
. . 导出了 "STUDY"."ORDER_DETAILS":"PARTITION_BEFORE_2017"  426.1 KB   15000 行
. . 导出了 "STUDY"."ORDER_DETAILS":"PARTITION_BEFORE_2018"      0 KB       0 行
已成功加载/卸载了主表 "STUDY"."SYS_EXPORT_SCHEMA_01" 
******************************************************************************
STUDY.SYS_EXPORT_SCHEMA_01 的转储文件集为:
  /home/oracle/expdir/study.dmp
作业 "STUDY"."SYS_EXPORT_SCHEMA_01" 已于 星期一 11月 25 10:44:55 2019 elapsed 0 00:01:14 成功完成
```

2、自动备份

```sql
#rman_level0.sh 
#!/bin/sh

export NLS_LANG='SIMPLIFIED CHINESE_CHINA.AL32UTF8'
export ORACLE_HOME=/home/oracle/app/oracle/product/12.1.0/dbhome_1  
export ORACLE_SID=orcl  
export PATH=$ORACLE_HOME/bin:$PATH  


rman target / nocatalog msglog=/home/oracle/rman_backup/lv0_`date +%Y%m%d-%H%M%S`_L0.log << EOF
run{
configure retention policy to redundancy 1;
configure controlfile autobackup on;
configure controlfile autobackup format for device type disk to '/home/oracle/rman_backup/%F';
configure default device type to disk;
crosscheck backup;
crosscheck archivelog all;
allocate channel c1 device type disk;
backup as compressed backupset incremental level 0 database format '/home/oracle/rman_backup/dblv0_%d_%T_%U.bak'
   plus archivelog format '/home/oracle/rman_backup/arclv0_%d_%T_%U.bak';
report obsolete;
delete noprompt obsolete;
delete noprompt expired backup;
delete noprompt expired archivelog all;
release channel c1;
}
EOF

exit
```


```text
#rman_level1.sh 
#!/bin/sh

export NLS_LANG='SIMPLIFIED CHINESE_CHINA.AL32UTF8'
export ORACLE_HOME=/home/oracle/app/oracle/product/12.1.0/dbhome_1  
export ORACLE_SID=orcl  
export PATH=$ORACLE_HOME/bin:$PATH  


rman target / nocatalog msglog=/home/oracle/rman_backup/lv1_`date +%Y%m%d-%H%M%S`_L0.log << EOF
run{
configure retention policy to redundancy 1;
configure controlfile autobackup on;
configure controlfile autobackup format for device type disk to '/home/oracle/rman_backup/%F';
configure default device type to disk;
crosscheck backup;
crosscheck archivelog all;
allocate channel c1 device type disk;
backup as compressed backupset incremental level 1 database format '/home/oracle/rman_backup/dblv1_%d_%T_%U.bak'
   plus archivelog format '/home/oracle/rman_backup/arclv1_%d_%T_%U.bak';
report obsolete;
delete noprompt obsolete;
delete noprompt expired backup;
delete noprompt expired archivelog all;
release channel c1;
}
EOF

exit
```

# 任务4：容灾,DataGuard

## 备库操作

1. 基础配置

| Type           | Primary         | Standby         |
| -------------- | --------------- | --------------- |
| IP地址           | 192.168.255.130 | 192.168.255.131 |
| db_name        | orcl            | orcl            |
| sid            | orcl            | orcl            |
| db_unique_name | orcl            | stdorcl         |

1. 在备库Standby中创建以下文件：

```text

mkdir -p /home/oracle/app/oracle/diag/orcl
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdborcl
mkdir -p /home/oracle/arch
mkdir -p /home/oracle/rman
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdbseed/
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdb/

```

结果：

![](https://tcs.teambition.net/storage/111neeb2a7821c3ca5eb438a48e3f9446873?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTU3NTI1NTMwMywiaWF0IjoxNTc0NjUwNTAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMW5lZWIyYTc4MjFjM2NhNWViNDM4YTQ4ZTNmOTQ0Njg3MyJ9.FcSvl7o_dgyJlkVQ3C3OjorbhLEZc8bmJscxJAMKMpw&download=image.png "")

1. 删除原有的数据库

```text

$sqlplus / as sysdba
shutdown immediate;
startup mount exclusive restrict;
drop database;

```

1. 将数据库启动到nomount状态

```text

$sqlplus / as sysdba
startup nomount

```

## 主库操作

```text

$sqlplus /  sysdba
select group#,thread#,members,status from v$log;

alter database add standby logfile  group 5 '/home/oracle/app/oracle/oradata/orcl/stdredo1.log' size 50m;
alter database add standby logfile  group 6 '/home/oracle/app/oracle/oradata/orcl/stdredo2.log' size 50m;
alter database add standby logfile  group 7 '/home/oracle/app/oracle/oradata/orcl/stdredo3.log' size 50m;
alter database add standby logfile  group 8 '/home/oracle/app/oracle/oradata/orcl/stdredo4.log' size 50m;

```

1. 主库环境强制开启归档

```text

ALTER DATABASE FORCE LOGGING;

alter system set LOG_ARCHIVE_CONFIG='DG_CONFIG=(orcl,stdorcl)' scope=both sid='*';         
alter system set log_archive_dest_1='LOCATION=/home/oracle/arch VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=orcl' scope=spfile;
alter system set LOG_ARCHIVE_DEST_2='SERVICE=stdorcl LGWR ASYNC  VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=stdorcl' scope=both sid='*';
alter system set fal_client='orcl' scope=both sid='*';    
alter system set FAL_SERVER='stdorcl' scope=both sid='*';  
alter system set standby_file_management=AUTO scope=both sid='*';
alter system set DB_FILE_NAME_CONVERT='/home/oracle/app/oracle/oradata/stdorcl/','/home/oracle/app/oracle/oradata/orcl/' scope=spfile sid='*';  
alter system set LOG_FILE_NAME_CONVERT='/home/oracle/app/oracle/oradata/stdorcl/','/home/oracle/app/oracle/oradata/orcl/' scope=spfile sid='*';
alter system set log_archive_format='%t_%s_%r.arc' scope=spfile sid='*';
alter system set remote_login_passwordfile='EXCLUSIVE' scope=spfile;
alter system set PARALLEL_EXECUTION_MESSAGE_SIZE=8192 scope=spfile;
```

## 编辑主库与备库

编辑文件:/home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora

*(这里的IP地址是你自己)*

```text

$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora

ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.255.130)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )

stdorcl =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.255.131)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SID = orcl)
    )
  )

```

1. 回到主库，在主库上生成备份库的参数文件

```text

SQL>create pfile from spfile;

```

1. 将主库的参数文件，密码文件拷贝到备库

```text

scp /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora 192.168.255.131:/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/
scp /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/orapworcl 192.168.255.131:/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/

```

1. 将主库复制到备库

```text

run{ 
allocate channel c1 type disk;
allocate channel c2 type disk;
allocate channel c3 type disk;
allocate AUXILIARY channel c4 type disk;
allocate AUXILIARY channel c5 type disk;
allocate AUXILIARY channel c6 type disk;
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  NOFILENAMECHECK;
release channel c1;
release channel c2;
release channel c3;
release channel c4;
release channel c5;
release channel c6;
}

```

# 备库操作

1. 在备库上更改参数文件

```text

$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora

```

文件内容是：

```text

orcl.__data_transfer_cache_size=0
orcl.__db_cache_size=671088640
orcl.__java_pool_size=16777216
orcl.__large_pool_size=33554432
orcl.__oracle_base='/home/oracle/app/oracle'#ORACLE_BASE set from environment
orcl.__pga_aggregate_target=536870912
orcl.__sga_target=1258291200
orcl.__shared_io_pool_size=50331648
orcl.__shared_pool_size=301989888
orcl.__streams_pool_size=0
*._allow_resetlogs_corruption=TRUE
*._catalog_foreign_restore=FALSE
*.audit_file_dest='/home/oracle/app/oracle/admin/orcl/adump'
*.audit_trail='db'
*.compatible='12.1.0.2.0'
*.control_files='/home/oracle/app/oracle/oradata/orcl/control01.ctl','/home/oracle/app/oracle/fast_recovery_area/orcl/control02.ctl','/home/oracle/app/oracle/fast_recovery_area/orcl/control03.ctl'
*.db_block_size=8192
*.db_domain=''
*.db_file_name_convert='/home/oracle/app/oracle/oradata/orcl/','/home/oracle/app/oracle/oradata/stdorcl/'
*.db_name='orcl'
*.db_unique_name='stdorcl'
*.db_recovery_file_dest='/home/oracle/app/oracle/fast_recovery_area'
*.db_recovery_file_dest_size=4823449600
*.diagnostic_dest='/home/oracle/app/oracle'
*.dispatchers='(PROTOCOL=TCP)(dispatchers=1)(pool=on)(ticks=1)(connections=500)(sessions=1000)'
*.enable_pluggable_database=true
*.fal_client='stdorcl'
*.fal_server='orcl'
*.inmemory_max_populate_servers=2
*.inmemory_size=157286400
*.local_listener=''
*.log_archive_config='DG_CONFIG=(stdorcl,orcl)'
*.log_archive_dest_1='LOCATION=/home/oracle/arch VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=stdorcl'
*.log_archive_dest_2='SERVICE=orcl LGWR ASYNC  VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=orcl'
*.log_archive_format='%t_%s_%r.arc'
*.log_file_name_convert='/home/oracle/app/oracle/oradata/orcl/','/home/oracle/app/oracle/oradata/stdorcl/'
*.max_dispatchers=5
*.max_shared_servers=20
*.open_cursors=400
*.parallel_execution_message_size=8192
*.pga_aggregate_target=511m
*.processes=300
*.recovery_parallelism=0
*.remote_login_passwordfile='EXCLUSIVE'
*.service_names='ORCL'
*.sga_max_size=1572864000
*.sga_target=1258291200
*.shared_server_sessions=200
*.standby_file_management='AUTO'
*.undo_tablespace='UNDOTBS1'

```

1. 在备库增加静态监听 

```text

$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/listener.ora

```

```text

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (ORACLE_HOME = /home/oracle/app/oracle/product/12.1.0/db_1)
      (SID_NAME = orcl)
    )
  )

```

1. 重新启动，备库开启实时应用模式

```text

$sqlplus / as sysdba
shutdown immediate
startup
alter database recover managed standby database disconnect;

```

## Dataguard测试

![](https://tcs.teambition.net/storage/111n188dc26d040c8b606099c6d336b374ce?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTU3NTMwNzA5MiwiaWF0IjoxNTc0NzAyMjkyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMW4xODhkYzI2ZDA0MGM4YjYwNjA5OWM2ZDMzNmIzNzRjZSJ9.szgtPE2segKNcV6lMwZTI3_01RUt-hHQszBjo_P5KB8&download=image.png "")

