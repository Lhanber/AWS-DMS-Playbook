## Importing Using Oracle Data Pump

Oracle Data Pump is a long-term replacement for the Oracle Export/Import utilities. Oracle Data Pump is the preferred way to move large amounts of data from an Oracle installation to an Amazon RDS DB instance. You can use Oracle Data Pump for several scenarios:

- Import data from an Oracle database (either on-premises or Amazon EC2 instance) to an Amazon RDS for Oracle DB instance.

- Import data from an RDS Oracle DB instance to an Oracle database (either on-premises or Amazon EC2 instance).

- Import data between RDS Oracle DB instances (for example, to migrate data from EC2-Classic to VPC).

To download Oracle Data Pump utilities, see [Oracle Database Software Downloads](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html) on the Oracle Technology Network website.

For compatibility considerations when migrating between versions of Oracle Database, see [the Oracle documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/oracle-data-pump-overview.html#GUID-BAA3B679-A758-4D55-9820-432D9EB83C68)

When you import data with Oracle Data Pump, you must transfer the dump file that contains the data from the source database to the target database. You can transfer the dump file using an Amazon S3 bucket or by using a database link between the two databases.

The following are best practices for using Oracle Data Pump to import data into an Amazon RDS for Oracle DB instance:

- Perform imports in schema or table mode to import specific schemas and objects.

- Limit the schemas you import to those required by your application.

- Do not import in full mode.

- Because Amazon RDS for Oracle does not allow access to SYS or SYSDBA administrative users, importing in full mode, or importing schemas for Oracle-maintained components, might damage the Oracle data dictionary and affect the stability of your database.

- When loading large amounts of data, transfer the dump file to the target Amazon RDS for Oracle DB instance, take a DB snapshot of your instance, and then test the import to verify that it succeeds. If database components are invalidated, you can delete the DB instance and re-create it from the DB snapshot. The restored DB instance includes any dump files staged on the DB instance when you took the DB snapshot.

- Do not import dump files that were created using the Oracle Data Pump export parameters TRANSPORT_TABLESPACES, TRANSPORTABLE, or TRANSPORT_FULL_CHECK. Amazon RDS for Oracle DB instances do not support importing these dump files.

## Migration methodology

Import data into Oracle database have two recommend options:

1. Import datapump file meta data only and use DMS full load + CDC to migrate data

2. Import datapump file meta data only first later import  data only and sepecify SCN in AWS DMS CDC Mode

The example in this section show one way to import data into an Oracle database by option 1

**Using Data Pump utilities impdp to import data**

1. Create the DB user which need to be imported
```
create user user_name identified by password
default tablespace users;
```

2. impdp command sample

```
impdp user/password@RDS_endpoint/SID DIRECTORY=DATA_PUMP_DIR CONTENT=METADATA_ONLY SCHEMAS=SCHEMA_NAME  REMAP_TABLESPACE=TABLESPCE_NAME:USERS dumpfile=export.dmp > export.log 2>&1
```

The sample impdp command above remap tablespce name to default tablespace named USERS,or you can create tablespace same as source DB,then you don't need to remap tablespace.

Sample command to list source DB user's tablespace,please see below:
```
SELECT USERNAME,DEFAULT_TABLESPACE FROM dba_users where USERNAME='user_name';
```

**Verify DB objects number**

Check if the number is same as source DB
```
SELECT DISTINCT OBJECT_TYPE , COUNT(*)
FROM DBA_OBJECTS
WHERE OWNER IN ('user1','user2','user3','user4','user5','user6')
GROUP BY OBJECT_TYPE ;
```
