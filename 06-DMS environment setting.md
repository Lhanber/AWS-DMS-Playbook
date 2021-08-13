## Before Start DMS Full load + CDC data migration
1. You have to check if there have any Temp table in migrated schema,please remember Temp table is not suitable for CDC Mode
2. You have to disable foreign key and trigger on the target DB before starting DMS task,after finished data migrate,please remember to enable foreign key and trigger.

Below is the SQL to disable foreign key and trigger.

Disable FK constraint
```
select 'ALTER TABLE '||cur.owner||'.'||cur.table_name||' MODIFY CONSTRAINT '||cur.constraint_name||' DISABLE;' from all_constraints cur where owner IN ('user1','user2','user3','user4') and status = 'ENABLED' and CONSTRAINT_TYPE='R';
```

Disable trigger
```
select 'alter trigger '||A.owner||'.'||A.trigger_name||' disable;' from all_triggers A where A.owner in ('user1','user2','user3','user4') order by A.owner, A.table_name;
Or
ALTER TABLE tablename DISABLE ALL TRIGGERS;
```


## Creating Tasks for Ongoing Replication Using AWS DMS

You can create an AWS DMS task that captures ongoing changes to the source data store. You can do this capture while you are migrating your data. You can also create a task that captures ongoing changes after you complete your initial (full-load) migration to a supported target data store. This process is called ongoing replication or change data capture (CDC). AWS DMS uses this process when replicating ongoing changes from a source data store. This process works by collecting changes to the database logs using the database engine's native API.

For Oracle, AWS DMS uses either the Oracle LogMiner API or binary reader API (bfile API) to read ongoing changes. AWS DMS reads ongoing changes from the online or archive redo logs based on the system change number (SCN).

There are two types of ongoing replication tasks:

- Full load plus CDC – The task migrates existing data and then updates the target database based on changes to the source database.

- CDC only – The task migrates ongoing changes after you have data on your target database.

This section will demonstrate Full load plus CDC.

**Create a Replication Instance**

Your first task in migrating a database is to create a replication instance that has sufficient storage and processing power to perform the tasks you assign and migrate data from your source database to the target database. The required size of this instance varies depending on the amount of data you need to migrate and the tasks that you need the instance to perform. For more information about replication instances, see [Working with an AWS DMS Replication Instance](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_ReplicationInstance.html).

Please see AWS online document to [create Replication Instance](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.html) step by TRANSPORT_TABLESPACES

**Specify Source and Target Endpoints**

While your replication instance is being created, you can specify the source and target data stores. The source and target data stores can be on an Amazon Elastic Compute Cloud (Amazon EC2) instance, an Amazon Relational Database Service (Amazon RDS) DB instance, or an on-premises database.

Please see AWS online document to [create source DB and target DB endpoint](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.html)

> Note 1:
In this playbook,we specify addSupplementalLogging=Y in target endpoint's Extra connection attributes, addSupplementalLogging=Y will automatically add supplemental logging if you enable this option.

> Note 2:
Target DB endpoints user must set uppercase, because Oracle default Characters is uppercase, if you set lowercase in target DB endpoints user,the dms task will show failed.

**Create a Task**

Create a task to specify what tables to migrate, to map data using a target schema, and to create new tables on the target database. As part of creating a task, you can choose the type of migration: to migrate existing data, migrate existing data and replicate ongoing changes, or replicate data changes only.

In this section,we choose migrate existing data and replicate ongoing changes.

Please see AWS online document to [create DMS task](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.html)

> Note:
Remember to set transformation rules,if you don't specify schema name in transformation rules,all tables include in Selection rules will load in to the user who you specify in target endpoint user.
For example,if your target endpoint user is Admin,and if you don't set transformation rules all tables include in Selection rules will load in to admin,so you have to set transformation rules rename option.
For example set below JSON to rename schema name:
```
{
    "rules": [
        {
            "rule-type": "transformation",
            "rule-id": "1",
            "rule-name": "1",
            "rule-target": "schema",
            "object-locator": {
                "schema-name": "schema_name"
            },
            "rule-action": "rename",
            "value": "schema_name",
            "old-value": null
```

Once you have finished with the task settings, choose **Create task**.

**Check DMS task cloudwatch log**

After you start DMS task,you can go to DMS cloudwatch log to see if there have any error message.

If you open DMS cloudwatch log,and encounter "Can't found cloudwatch log group" error,please follow [Why can't I see CloudWatch Logs for an AWS DMS task?](https://aws.amazon.com/tw/premiumsupport/knowledge-center/dms-cloudwatch-logs-not-appearing/) to troubleshoot.
