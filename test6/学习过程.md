## 期末项目
### 表设计
* 管理员表
```

```
### 设计项目涉及的表及表空间使用方案。至少5张表和5万条数据，两个表空间。
数据库存储管理-表空间学习
6.2 创建表空间_删除表空间_查询表空间
```
sqlplus c##user1/123@pdborcl
create TABLESPACE users02 DATAFILE '/home/oracle/app/oracle/oradata/orcla/pdborcl/pdbtest_users02_1.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED, '/home/oracle/app/oracle/oradata/orcla/pdborcl/pdbtest_users02_2.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
drop TABLESPACE users02 INCLUDING CONTENTS AND DATAFILES;
select tablespace_name status,contents,logging from dba_tablespaces;
```

### 用户及权限管理:至少两类角色，两个用户
* 创建三类角色
用户及权限管理-学习
7.2.2创建工共角色
```
sqlplus / as sysdba
col role format a20
col privilege format a20
col common format a7
select *from role_sys_privs;
SELECT * FROM role_sys_privs WHERE role='CONNECT';
create role c##con_res;
grant set container, create session, create table, create cluster, create operator, create indextype, create type, create sequence, create trigger, create procedure to c##con_res container = all;
select * from dba_sys_privs where grantee='C##CON_RES';
select * from dba_role_privs where grantee='C##CON_RES';
select * from dba_roles where role='C##CON_RES';
ALTER SESSION SET CONTAINER=pdborcl;
alter database open;
SELECT *FROM dba_roles WHERE role='C##CON_RES';
SELECT *FROM dba_role_privs WHERE grantee='C##CON_RES';
```
7.2.3创建本地角色
```
sqlplus / as sysdba
alter session set container=pdborcl;
alter pluggable database open;
select name,open_mode from v$pdbs;
CREATE ROLE con_res;
grant connect,resource to con_res;

col grantee format a20
col common format a7
col role FORMAT a20
SELECT *FROM dba_roles WHERE role='CON_RES';

SELECT *FROM dba_role_privS WHERE grantee='CON_RES';
SELECT *FROM dba_sys_privs WHERE grantee='CON_RES';
DROP ROLE con_res;
```
用户管理
7.3.1 创建工共用户
```
sqlplus / as sysdba
CREATE USER c##user1 IDENTIFIED BY 123 CONTAINER=ALL;

COL username FORMAT a20
SELECT username,con_id FROM cdb_users WHERE username='C##USER1';
GRANT c##con_res TO c##user1 CONTAINER=ALL;
ALTER USER c##user1 DEFAULT ROLE C##CON_RES CONTAINER=ALL;

col granted_role format a15
col granted format a15
SELECT *FROM cdb_role_privs WHERE grantee='C##USER1';

ALTER USER c##user1 DEFAULT TABLESPACE system CONTAINER=ALL;
ALTER USER c##user1 QUOTA 100M ON system;

sqlplus c##user1/123@pdborcl

col role format a40
select *from session_roles;

select *from session_privs;

CREATE TABLE t1(id number);
INSERT INTO t1 (id) VALUES (1);
col table_name format a20
col tablespace_name format a20
SELECT table_name, tablespace_name FROM user_tables;
```
7.3.2 授予用户对象权限
```
见书：159
```

任务1：新键一个PDB,管理员为adim 123
```
CREATE PLUGGABLE DATABASE weibo ADMIN USER admin IDENTIFIED BY 123 FILE_NAME_CONVERT=('home/oracle/app/oracle/oradata/orcl/pdbseed','home/oracle/app/oracle/oradata/orcl/weobo');
```