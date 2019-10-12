# 试验二
* 第1步：以system登录到pdborcl，创建角色con_cmq_view和用户cmq_user，并授权和分配空间：
```
执行 CREATE ROLE con_cmq_view;
结果 ：
Role CON_CMQ_VIEW 已创建。
```
```
执行 ：GRANT connect,resource,CREATE VIEW TO con_cmq_view;
结果 ：
Grant 成功。
```
```
执行 ：CREATE USER cmq_user IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
结果 ：
User CMQ_USER 已创建。
```

```
执行 ：ALTER USER cmq_user QUOTA 50M ON users;
结果 ：
User CMQ_USER已变更。
```

```
执行 ：GRANT con_cmq_view TO cmq_user;
结果 ：
Grant 成功。
```
* 第2步：新用户new_user连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
```
执行 ：show user;
结果 ：
USER 是 "CMQ_USER"
```

```
执行 ：CREATE TABLE mytable (id number,name varchar(50));
结果 ：
Table MYTABLE 已创建。
```

```
执行 ：INSERT INTO mytable(id,name)VALUES(1,'zhang');
结果 ：
1行已插入。
```

```
执行 ：INSERT INTO mytable(id,name)VALUES (2,'wang');
结果 ：
1行已插入。
```

```
执行 ：CREATE VIEW myview AS SELECT name FROM mytable;
View created.
结果 ：
View MYVIEW 已创建。
```

```
执行 ：SELECT * FROM myview;
结果 ：
NAME                                              
--------------------------------------------------
zhang
wang
```

```
执行 ：GRANT SELECT ON myview TO hr;
结果 ：
Grant 成功。
```
* 第三步，登陆到hr查看myview
```
执行 ：show user;
结果 ：USER 是 "HR"

```

```
执行 ：SELECT * FROM cmq_user.myview;
结果 ：
NAME                                              
--------------------------------------------------
zhang
wang
```
* 第四步，授权SELECT给其他同学:
```
执行 ：GRANT SELECT ON myview TO jet5devil;
结果 ：
Grant 成功。
```
* 第五步，查询同学给我的权限
```
执行 ：SELECT * FROM jet5devil.myview;
结果 ：
--------------------------------------------------
zhang
wang
```