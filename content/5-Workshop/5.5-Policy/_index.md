---
title: "5. Cost Saving Automation: Lambda + EventBridge"
weight: 5
date: 2026-08-05
draft: false
---

## 5.1. IAM Role for Lambda

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["ec2:StartInstances", "ec2:StopInstances", "ec2:DescribeInstances"],
            "Resource": "arn:aws:ec2:ap-southeast-1:<account-id>:instance/<instance-id>"
        },
        {
            "Effect": "Allow",
            "Action": ["rds:StartDBInstance", "rds:StopDBInstance", "rds:DescribeDBInstances"],
            "Resource": "arn:aws:rds:ap-southeast-1:<account-id>:db:<db-instance-id>"
        },
        {
            "Effect": "Allow",
            "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```

## 5.2. Lambda Function

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

**Important**: Increase the default Timeout from 3 seconds to 30 seconds (**Configuration → General configuration**) — calling two consecutive AWS APIs typically takes more than 3 seconds.

## 5.3. EventBridge Scheduler — 4 Schedules

| Schedule | Time (VN, UTC+7) | Cron | Input |
|---|---|---|---|
| `plantify-start-morning` | 08:00 | `0 8 * * ? *` | `{"action": "start"}` |
| `plantify-stop-evening` | 17:30 | `30 17 * * ? *` | `{"action": "stop"}` |
| `plantify-start-night` | 20:00 | `0 20 * * ? *` | `{"action": "start"}` |
| `plantify-stop-midnight` | 00:00 | `0 0 * * ? *` | `{"action": "stop"}` |

For each schedule: Target = AWS Lambda Invoke → Select function → Input = Constant JSON text as specified in the table above.

## Common Issues

| Issue | Cause | Solution |
|---|---|---|
| `Sandbox.Timedout` when testing Lambda | The default 3-second timeout is too short | Increase to 30 seconds under General configuration |
| Schedules execute at unexpected times | Confusion between UTC and local time — the Timezone field in EventBridge Scheduler defaults to local time (UTC+07:00), so manual UTC conversion is unnecessary | Check the Timezone field before setting up cron expressions and enter local time directly in the UI |
| Calling `stop` when resources are already off throws an error | The RDS API is not idempotent like EC2 — throwing `InvalidDBInstanceStateFault` if called on an already stopped state | Wrap in a `try/except` block to catch and ignore this exception, preventing script failure |