# Using an Oracle Database as a Source for AWS DMS

You can migrate data from one or many Oracle databases using AWS DMS. With an Oracle database as a source, you can migrate data to any of the targets supported by AWS DMS.

DMS supports the following Oracle database editions:

- Oracle Enterprise Edition

- Oracle Standard Edition

- Oracle Express Edition

- Oracle Personal Edition

For self-managed Oracle databases as sources, AWS DMS supports all Oracle database editions for versions 10.2 and later, 11g and up to 12.2, 18c, and 19c. For Amazon-managed Oracle databases provided by Amazon RDS, AWS DMS supports all Oracle database editions for versions 11g (versions 11.2.0.3.v1 and later) and up to 12.2, 18c, and 19c.

# Working with a Self-Managed Oracle Database as a Source for AWS DMS

To use an Oracle database as a source in AWS DMS, grant the privileges following to the Oracle user specified in the Oracle endpoint connection settings.

    GRANT SELECT on V_$ARCHIVED_LOG to dms_user;
    GRANT SELECT on V_$LOG to dms_user;
    GRANT SELECT on V_$LOGFILE to dms_user;
    GRANT SELECT on V_$DATABASE to dms_user;
    GRANT SELECT on V_$THREAD to dms_user;
    GRANT SELECT on V_$PARAMETER to dms_user;
    GRANT SELECT on V_$NLS_PARAMETERS to dms_user;
    GRANT SELECT on V_$TIMEZONE_NAMES to dms_user;
    GRANT SELECT on V_$TRANSACTION to dms_user;
    GRANT SELECT on ALL_INDEXES to dms_user;
    GRANT SELECT on ALL_OBJECTS to dms_user;
    GRANT SELECT on DBA_OBJECTS to dms_user; (required if the Oracle version is earlier than 11.2.0.3)
    GRANT SELECT on ALL_TABLES to dms_user;
    GRANT SELECT on ALL_USERS to dms_user;
    GRANT SELECT on ALL_CATALOG to dms_user;
    GRANT SELECT on ALL_CONSTRAINTS to dms_user;
    GRANT SELECT on ALL_CONS_COLUMNS to dms_user;
    GRANT SELECT on ALL_TAB_COLS to dms_user;
    GRANT SELECT on ALL_IND_COLUMNS to dms_user;
    GRANT SELECT on ALL_LOG_GROUPS to dms_user;
    GRANT SELECT on SYS.DBA_REGISTRY to dms_user;
    GRANT SELECT on SYS.OBJ$ to dms_user;
    GRANT SELECT on DBA_TABLESPACES to dms_user;
    GRANT SELECT on ALL_TAB_PARTITIONS to dms_user;
    GRANT SELECT on ALL_ENCRYPTED_COLUMNS to dms_user;
    GRANT SELECT on V_$LOGMNR_LOGS to dms_user;
    GRANT SELECT on V_$LOGMNR_CONTENTS to dms_user;
    GRANT SELECT on V_$STANDBY_LOG to dms_user;

## Account Privileges Required When Using Oracle LogMiner to Access the Redo Logs

- CREATE SESSION

- EXECUTE on DBMS_LOGMNR

- SELECT on V_$LOGMNR_LOGS

- SELECT on V_$LOGMNR_CONTENTS

- GRANT LOGMINING – Required only if the Oracle version is 12c or later.

## Configuring a Self-Managed Oracle Source for Replication with AWS DMS

**Setting Logs to ARCHIVELOG Mode**

    ALTER database ARCHIVELOG;

**Verifying and Setting Up Supplemental Logging**

1. Verify that supplemental logging is enabled for the database.

2. Verify that the required supplemental logging is enabled for each table.

3. If a filter or transformation is defined for a table, enable additional logging as needed.

**To verify and, if needed, enable supplemental logging for the database**

1. Run the sample query following to verify that the current version of the Oracle database is supported by AWS DMS. If the query runs without error, the returned database version is supported.

 ```SELECT name, value, description FROM v$parameter WHERE name = compatible;```

Here, name, value, and description are columns somewhere in the database that are being queried based on the value of name. As part of the query, an AWS DMS task checks the value of v$parameter against the returned version of the Oracle database. If there is a match, the query runs without error and AWS DMS supports this version of the database. If there isn't a match, the query raises an error and AWS DMS doesn't support this version of the database. In that case, to proceed with migration, first convert the Oracle database to a AWS DMS-supported version.

2. Run the query following to verify that supplemental logging is enabled for the database. If the returned result is YES or IMPLICIT, supplemental logging is enabled for the database.

```SELECT supplemental_log_data_min FROM v$database;```

3. If needed, enable supplemental logging for the database by running the command following.

```ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;```
