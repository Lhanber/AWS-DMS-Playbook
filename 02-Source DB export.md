# Before export DB
1. Check source DB size for estimate migration time
**Use below SQL to check DB size**
```
select sum(bytes)/1024/1024 size_in_mb from dba_segments;
```
2. Check lob column PK or UK

Currently, a table must have a primary key for AWS DMS to capture LOB changes. If a table that contains LOBs doesn't have a primary key, there are several actions you can take to capture LOB changes:
- Add a primary key to the table. This can be as simple as adding an ID column and populating it with a sequence using a trigger.

- Create a materialized view of the table that includes a system generated ID as the primary key and migrate the materialized view rather than the table.

- Create a logical standby, add a primary key to the table, and migrate from the logical standby.

**Use below SQL to list table with LOB column and don't have PK or UK**
```
SELECT A1.TABLE_NAME
FROM (
	SELECT TABLE_NAME FROM ALL_TAB_COLS WHERE OWNER='UATUDESIGNNEW' AND ALL_TAB_COLS.DATA_TYPE IN ('CLOB', 'LOB', 'BLOB')) A1
INNER JOIN
(SELECT TABLE_NAME FROM ALL_TABLES WHERE OWNER='UATUDESIGNNEW'
MINUS
SELECT TABLE_NAME FROM ALL_CONSTRAINTS WHERE OWNER='UATUDESIGNNEW' AND CONSTRAINT_TYPE IN ('P','U'))
A2 ON A1.TABLE_NAME = A2.TABLE_NAME;
```

3. Check lob column size for setting DMS LOB Mode
**Use below SQL to list all LOB column with PK or UK**
```
SELECT 'select sum(dbms_lob.getlength('|| ALL_LOBS.COLUMN_NAME ||')) from ' || ALL_LOBS.TABLE_NAME ||';'
FROM ALL_LOBS,
ALL_CONSTRAINTS,
ALL_TAB_COLS
WHERE ALL_LOBS.OWNER = ALL_TAB_COLS.OWNER
AND ALL_LOBS.OWNER=ALL_CONSTRAINTS.OWNER
AND all_constraints.constraint_type IN ('P','U')
AND ALL_TAB_COLS.DATA_TYPE IN ('CLOB', 'LOB', 'BLOB');
```
**Check LOB column size**
```
SELECT MAX(DBMS_LOB.getlength(column name)/1024) KB FROM table name;
```

# Export DB
AWS recommend export by schema level,so all we need is to export schemas which we need to migrate.

**Export DB via oracle datapump by schemas**
```
expdp user/password@connection_string DIRECTORY=DATA_PUMP_DIR schemas=user1,user2,user3,user4,user5,user6  dumpfile=export.dmp > export.log 2>&1
```

**Check export log if have any error**
```
vi export.log
```
