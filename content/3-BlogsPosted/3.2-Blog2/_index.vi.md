
---
title: "3 cái bẫy chi phí AWS mà tân binh dễ dính (và cách né bằng Free Tier + tự động hoá)"
date: 2026-08-04
draft: false
tags: ["aws", "cost-optimization", "free-tier"]
description: "Ba tình huống thực tế khiến chi phí AWS phát sinh ngoài dự kiến khi làm dự án cá nhân, và cách phòng tránh từ đầu."
---

Làm dự án AWS đầu tiên, nỗi lo lớn nhất không phải là "làm sao cho chạy" mà là "làm sao đừng bị trừ tiền oan". Dưới đây là 3 tình huống có thật đã gặp khi triển khai Plantify Co lên AWS, cùng cách xử lý.

## 1. Elastic IP — miễn phí, nhưng không phải mãi mãi

Nhiều tài liệu cũ (và cả AI, kể cả bản thân mình lúc đầu) hay nói "Elastic IP miễn phí khi đang gắn vào instance đang chạy". Điều này **không còn đúng hoàn toàn** từ 1/2/2024 — AWS đổi chính sách: **mọi địa chỉ IPv4 công khai đều tính phí** khoảng $0.005/giờ (~$3.65/tháng), bất kể đang gắn vào instance chạy hay không.

Tin đỡ hơn: Free Tier của EC2 bao gồm 750 giờ sử dụng IPv4 công khai/tháng cho 12 tháng đầu — đủ phủ hết 1 tháng chạy 24/7 với đúng 1 địa chỉ IP. Nghĩa là:

- Trong 12 tháng đầu, dùng đúng 1 Elastic IP: vẫn miễn phí.
- Sau 12 tháng, hoặc nếu có thêm IP thứ 2: bắt đầu tính phí.

**Bài học**: đừng tin tưởng tuyệt đối vào thông tin cũ về giá AWS — chính sách giá thay đổi khá thường xuyên, luôn kiểm tra lại trang pricing chính thức trước khi đưa ra quyết định kiến trúc dựa trên "cái gì miễn phí".

## 2. Tạo nhầm region cho S3 — không tốn tiền ngay, nhưng ảnh hưởng tốc độ và khó giải trình

Khi tạo bucket S3 để backup database, mình vô tình để mặc định ở **US East (N. Virginia)** trong khi EC2 và RDS đều ở **Singapore**. Không sai kỹ thuật — S3 vẫn truy cập được xuyên region — nhưng:

- Data phải đi từ Singapore sang Virginia mỗi lần backup, tăng độ trễ và tăng nhẹ phí truyền dữ liệu.
- Khi trình bày kiến trúc trong báo cáo, khó giải thích hợp lý lý do vì sao S3 lại khác region với phần còn lại, ngoài "làm nhầm".

**Bài học**: luôn kiểm tra kỹ dropdown Region **trước khi** bấm Create — S3 không cho đổi region sau khi tạo, phải xoá và tạo lại từ đầu nếu sai.

## 3. Chạy EC2 + RDS 24/7 dù không cần thiết

Với một dự án demo/học tập, không cần server chạy suốt ngày đêm. Giải pháp: dùng **AWS Lambda + EventBridge Scheduler** để tự động Stop tài nguyên ngoài giờ sử dụng và Start lại đúng giờ cần dùng.

```python
def lambda_handler(event, context):
    action = event.get('action')
    ec2 = boto3.client('ec2', region_name=REGION)
    rds = boto3.client('rds', region_name=REGION)

    if action == 'start':
        ec2.start_instances(InstanceIds=[EC2_INSTANCE_ID])
        rds.start_db_instance(DBInstanceIdentifier=RDS_INSTANCE_ID)
    elif action == 'stop':
        ec2.stop_instances(InstanceIds=[EC2_INSTANCE_ID])
        rds.stop_db_instance(DBInstanceIdentifier=RDS_INSTANCE_ID)
```

Kết hợp 4 lịch trình EventBridge (bật sáng, tắt chiều tối muộn, bật tối, tắt nửa đêm), tài nguyên chỉ chạy khoảng 13-14 giờ/ngày thay vì 24 giờ — sau khi hết Free Tier, đây là khoản tiết kiệm đáng kể, và ngay cả trong Free Tier, đây vẫn là thói quen tốt cần luyện tập sớm.

## Lưới an toàn nên thiết lập ngay từ ngày đầu

Trước khi tạo bất kỳ resource nào, nên bật:

- **AWS Budgets → Zero spend budget**: cảnh báo ngay khi phát sinh bất kỳ khoản phí nào.
- **Free Tier usage alerts**: cảnh báo khi dùng gần hết hạn mức miễn phí.

Hai thứ này không tốn công thiết lập (vài phút), nhưng là lưới an toàn tốt nhất để tránh việc "mở mắt dậy thấy bill $50" khi có gì đó chạy sai.

## Tổng kết

Chi phí AWS không đáng sợ nếu hiểu rõ 3 điều: Free Tier áp dụng cho cái gì và trong bao lâu, luôn kiểm tra kỹ trước khi tạo resource (đặc biệt là region), và tự động hoá việc tắt/mở tài nguyên khi không cần chạy liên tục.
