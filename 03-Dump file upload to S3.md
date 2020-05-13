# Upload dump file to S3


1. Navigate to datapump DIRECTORY

  Please check your datapump command

EX:
```
expdp user/password@connection_string DIRECTORY=DATA_PUMP_DIR schemas=user1,user2,user3,user4,user5,user6  dumpfile=export.dmp
```
DATA_PUMP_DIR is the directory where your dump file located

Login to DB and Use below SQL to find DATA_PUMP_DIR path
```
SELECT * FROM ALL_DIRECTORIES;
```

2. Using AWS S3 CLI to upload to S3
Copy export.dmp in current directory to s3://my-bucket/path
```
$ aws s3 cp export.dmp s3://my-bucket/path/
```
Ref:

https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-services-s3-commands.html
