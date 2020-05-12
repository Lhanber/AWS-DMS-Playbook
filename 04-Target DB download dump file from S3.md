## Amazon S3 Integration

You can transfer files between an Amazon RDS for Oracle DB instance and an Amazon S3 bucket. You can use Amazon S3 integration with Oracle features such as Data Pump. For example, you can download Data Pump files from Amazon S3 to the DB instance host.

## Prerequisites for Amazon RDS Oracle Integration with Amazon S3

To work with Amazon RDS for Oracle integration with Amazon S3, the Amazon RDS DB instance must have access to an Amazon S3 bucket. For this, you create an AWS Identity and Access Management (IAM) policy and an IAM role. The Amazon VPC used by your DB instance doesn't need to provide access to the Amazon S3 endpoints.

**To grant Amazon RDS access to an Amazon S3 bucket**

1. Create an AWS Identity and Access Management (IAM) policy that grants Amazon RDS access to an Amazon S3 bucket.

Include the appropriate actions in the policy based on the type of access required:

- GetObject – Required to transfer files from an Amazon S3 bucket to Amazon RDS.
- ListBucket – Required to transfer files from an Amazon S3 bucket to Amazon RDS.
- PutObject – Required to transfer files from Amazon RDS to an Amazon S3 bucket.

The following AWS CLI command creates an IAM policy named rds-s3-integration-policy with these options. It grants access to a bucket named your-s3-bucket-arn.

**Example**

For Linux, macOS, or Unix:

```

aws iam create-policy \
   --policy-name rds-s3-integration-policy \
   --policy-document '{
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "s3integration",
         "Action": [
           "s3:GetObject",
           "s3:ListBucket",
           "s3:PutObject"
         ],
         "Effect": "Allow",
         "Resource": [
           "arn:aws:s3:::your-s3-bucket-arn",
           "arn:aws:s3:::your-s3-bucket-arn/*"
         ]
       }
     ]
   }'                        

```

2. After the policy is created, note the Amazon Resource Name (ARN) of the policy. You need the ARN for a subsequent step.

3. Create an IAM role that Amazon RDS can assume on your behalf to access your Amazon S3 buckets.

The following AWS CLI command creates the rds-s3-integration-role for this purpose.

**Example**

For Linux, macOS, or Unix:
```

aws iam create-role \
   --role-name rds-s3-integration-role \
   --assume-role-policy-document '{
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
            "Service": "rds.amazonaws.com"
          },
         "Action": "sts:AssumeRole"
       }
     ]
   }'                            

```

4. After the role is created, note the ARN of the role. You need the ARN for a subsequent step.

Attach the policy you created to the role you created.

The following AWS CLI command attaches the policy to the role named rds-s3-integration-role.

**Example**

For Linux, macOS, or Unix:

```

aws iam attach-role-policy \
   --policy-arn your-policy-arn \
   --role-name rds-s3-integration-role                             

```

6. Add the role to the Oracle DB instance.

The following AWS CLI command adds the role to an Oracle DB instance named mydbinstance.

**Example**

For Linux, macOS, or Unix:

```

aws rds add-role-to-db-instance \
   --db-instance-identifier mydbinstance \
   --feature-name S3_INTEGRATION \
   --role-arn your-role-arn                           

```

## Adding the Amazon S3 Integration Option

1. Create a new option group or identify an existing option group to which you can add the S3_INTEGRATION option.
2. Add the S3_INTEGRATION option to the option group.
3. Create a new Oracle DB instance and associate the option group with it, or modify an Oracle DB instance to associate the option group with it.

Ref:
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-s3-integration.html

## Transferring Files Between Amazon RDS for Oracle and an Amazon S3 Bucket

You can use Amazon RDS procedures to upload files from an Oracle DB instance to an Amazon S3 bucket. You can also use Amazon RDS procedures to download files from an Amazon S3 bucket to an Oracle DB instance.

**Downloading Files from an Amazon S3 Bucket to an Oracle DB Instance**

To download files from an Amazon S3 bucket to an Oracle DB instance, use the Amazon RDS procedure rdsadmin.rdsadmin_s3_tasks.download_from_s3.

The following example downloads all of the files in the Amazon S3 bucket named mys3bucket to the DATA_PUMP_DIR directory.

```

SELECT rdsadmin.rdsadmin_s3_tasks.download_from_s3(
      p_bucket_name    =>  'mys3bucket',       
      p_directory_name =>  'DATA_PUMP_DIR')
   AS TASK_ID FROM DUAL;            

```

**Monitoring the Status of a File Transfer**

You can view the status of an ongoing task in a bdump file. The bdump files are located in the /rdsdbdata/log/trace directory. Each bdump file name is in the following format.

```
dbtask-task-id.log            

```

You can use the rdsadmin.rds_file_util.read_text_file stored procedure to view the contents of bdump files. For example, the following query returns the contents of the dbtask-1546988886389-2444.log bdump file.

```

SELECT text FROM table(rdsadmin.rds_file_util.read_text_file('BDUMP','dbtask-1546988886389-2444.log'));            

```


**Check DATA_PUMP_DIR directory have dump file**
```
select * from table(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) order by mtime;

```

Ref:

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-s3-integration.html
