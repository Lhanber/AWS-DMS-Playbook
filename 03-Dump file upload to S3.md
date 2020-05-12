# Upload dump file to S3


1. navigate to datapump DIRECTORY

Please check your datapump command

EX:
```
expdp user/password@connection_string DIRECTORY=DATA_PUMP_DIR schemas=user1,user2,user3,user4,user5,user6  dumpfile=export.dmp
```
DATA_PUMP_DIR is where your dumpfile's location

2. Using AWS S3 CLI to upload to S3
Copy MyFile.txt in current directory to s3://my-bucket/path
```
$ aws s3 cp export.dmp s3://my-bucket/path/
```
Ref:

https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-services-s3-commands.html
