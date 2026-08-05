---
title: "5. Tự động hoá tiết kiệm chi phí: Lambda + EventBridge"
weight: 5
date: 2026-08-05
draft: false
---

## 5.1. IAM Role cho Lambda

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

**Quan trọng**: tăng Timeout mặc định từ 3 giây lên 30 giây (Configuration → General configuration) — gọi 2 API AWS liên tiếp thường mất hơn 3 giây.

## 5.3. EventBridge Scheduler — 4 lịch trình

| Lịch | Giờ (VN, UTC+7) | Cron | Input |
|---|---|---|---|
| `plantify-start-morning` | 08:00 | `0 8 * * ? *` | `{"action": "start"}` |
| `plantify-stop-evening` | 17:30 | `30 17 * * ? *` | `{"action": "stop"}` |
| `plantify-start-night` | 20:00 | `0 20 * * ? *` | `{"action": "start"}` |
| `plantify-stop-midnight` | 00:00 | `0 0 * * ? *` | `{"action": "stop"}` |

Mỗi lịch: Target = AWS Lambda Invoke → chọn function → Input = Constant JSON text theo bảng trên.

## Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| `Sandbox.Timedout` khi test Lambda | Timeout mặc định 3 giây quá ngắn | Tăng lên 30 giây trong General configuration |
| Lịch chạy sai giờ so với dự kiến | Nhầm lẫn giữa giờ UTC và giờ địa phương — field Timezone của EventBridge Scheduler mặc định có thể đã là giờ địa phương (UTC+07:00), không cần tự quy đổi UTC nữa | Kiểm tra field Timezone của lịch trước khi tính cron, dùng thẳng giờ hiển thị trên UI |
| Gọi `stop` khi tài nguyên đã tắt sẵn gây lỗi | RDS API không "idempotent" như EC2 — báo lỗi `InvalidDBInstanceStateFault` nếu gọi trùng trạng thái | Bọc `try/except` bắt lỗi này và bỏ qua, không cho crash |