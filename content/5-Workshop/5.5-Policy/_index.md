---

title: "5. Automating cost optimization with Lambda and EventBridge"
weight: 5
date: 2026-08-05
draft: false
------------

## 5.1. IAM role for Lambda

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:DescribeInstances"
            ],
            "Resource": "arn:aws:ec2:ap-southeast-1:<account-id>:instance/<instance-id>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "rds:StartDBInstance",
                "rds:StopDBInstance",
                "rds:DescribeDBInstances"
            ],
            "Resource": "arn:aws:rds:ap-southeast-1:<account-id>:db:<db-instance-id>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```

This role grants Lambda permission to start and stop a specific EC2 instance and RDS database instance, while also allowing it to write execution logs to CloudWatch Logs.

## 5.2. Lambda function

```python
import boto3

EC2_INSTANCE_ID = '<instance-id>'
RDS_INSTANCE_ID = '<db-instance-id>'
REGION = 'ap-southeast-1'

def lambda_handler(event, context):
    action = event.get('action')
    ec2 = boto3.client('ec2', region_name=REGION)
    rds = boto3.client('rds', region_name=REGION)

    if action == 'start':
        ec2.start_instances(InstanceIds=[EC2_INSTANCE_ID])
        try:
            rds.start_db_instance(DBInstanceIdentifier=RDS_INSTANCE_ID)
        except rds.exceptions.InvalidDBInstanceStateFault:
            pass
        return {'status': 'started'}

    elif action == 'stop':
        ec2.stop_instances(InstanceIds=[EC2_INSTANCE_ID])
        try:
            rds.stop_db_instance(DBInstanceIdentifier=RDS_INSTANCE_ID)
        except rds.exceptions.InvalidDBInstanceStateFault:
            pass
        return {'status': 'stopped'}

    return {'status': 'no action specified'}
```

A key configuration change is increasing the Lambda **Timeout** from the default **3 seconds** to **30 seconds**.

Because the function invokes both the EC2 and RDS APIs sequentially, the combined execution time often exceeds the default timeout.

Configuration path:

**Lambda → Configuration → General configuration → Timeout**

## 5.3. EventBridge Scheduler

Four schedules were created to automatically start and stop the application infrastructure during working hours.

<Table columnSizing=
