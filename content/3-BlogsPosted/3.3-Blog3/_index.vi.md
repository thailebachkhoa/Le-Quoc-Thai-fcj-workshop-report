---
title: "CloudWatch Alarm tưởng dễ mà không dễ: câu chuyện set ngược điều kiện Greater/Lower Than"
date: 2026-08-04
draft: false
tags: ["aws", "cloudwatch", "monitoring"]
description: "Một lỗi cấu hình nhỏ nhưng dễ mắc khi tạo CloudWatch Alarm giám sát dung lượng RDS, và cách phát hiện, sửa kịp thời."
---

## Bối cảnh

Sau khi triển khai RDS cho dự án Plantify Co, mình muốn có cảnh báo tự động khi dung lượng ổ đĩa database sắp hết — tránh tình huống database ngừng ghi được dữ liệu mà không ai biết trước. CloudWatch Alarm + SNS (gửi email) là công cụ hợp lý cho việc này.

## Cấu hình ban đầu — và cú sốc đầu tiên

Tạo Alarm theo dõi metric `FreeStorageSpace` của RDS, đặt ngưỡng cảnh báo khi dung lượng trống dưới 2GB. Vài phút sau khi bật, một email ập tới:

```
ALARM: "Plantify-RDS-LowStorage" in Asia Pacific (Singapore)
Reason: Threshold Crossed: 1.95011616768E10 was greater than the threshold (2.0E9)
```

Dịch lại: dung lượng trống hiện tại (~19.5GB) **lớn hơn** ngưỡng 2GB — và Alarm báo động vì điều kiện đang là... **"Greater than"** (lớn hơn) thay vì **"Lower than"** (thấp hơn). Nói cách khác, mình vô tình cấu hình: "báo động khi dung lượng trống VƯỢT QUÁ 2GB" — mà database gần như lúc nào cũng có nhiều hơn 2GB trống, nên Alarm kêu ngay lập tức dù chẳng có sự cố gì cả.

## Vì sao dễ mắc lỗi này

Khi tạo Alarm trong CloudWatch Console, phần "Conditions" cho bạn chọn giữa các operator: Greater than, Greater than or equal, Lower than, Lower than or equal. Với một số metric (như CPU cao là xấu → dùng Greater Than là trực giác), nhưng với metric khác (như dung lượng trống thấp là xấu → phải dùng Lower Than), việc chọn nhầm operator rất dễ xảy ra nếu không dừng lại suy nghĩ kỹ ý nghĩa của từng metric trước khi chọn điều kiện.

## Cách phát hiện và sửa

Đọc kỹ nội dung email cảnh báo — CloudWatch luôn ghi rõ giá trị thực tế, ngưỡng đặt ra, và operator đang dùng trong phần "Reason for State Change". Đây chính là chỗ phát hiện ra vấn đề: giá trị thực tế và ngưỡng không hề "xấu" theo nghĩa thông thường, chỉ là điều kiện so sánh bị đặt sai chiều.

Vào lại Alarm, sửa:
```
Trước: Whenever FreeStorageSpace is Greater than 2000000000
Sau:   Whenever FreeStorageSpace is Lower than 2000000000
```

Sau khi sửa, Alarm tự động chuyển về trạng thái **OK** trong vài phút — đây cũng là cách xác nhận Alarm hoạt động đúng cả 2 chiều: biết báo động khi có vấn đề, và biết tự phục hồi khi vấn đề không còn.

## Bài học rút ra

- **Luôn đọc kỹ nội dung email/log cảnh báo** thay vì chỉ nhìn thấy chữ "ALARM" màu đỏ rồi hoảng — phần "Reason for State Change" luôn cho biết chính xác điều gì đang xảy ra.
- **Test cả 2 chiều trước khi tin tưởng vận hành thật**: không chỉ kiểm tra Alarm có kích hoạt được không, mà còn phải kiểm tra nó có tự tắt (trở về OK) đúng khi điều kiện không còn nữa.
- Với mỗi metric, dừng lại 5 giây để tự hỏi: "giá trị cao là tốt hay xấu?" trước khi chọn Greater/Lower Than — với `FreeStorageSpace`, cao là tốt (còn nhiều chỗ trống) nên phải cảnh báo khi **thấp**; với `CPUUtilization`, cao là xấu (quá tải) nên cảnh báo khi **cao**. Nghe đơn giản nhưng rất dễ lẫn lộn lúc đang thao tác nhanh trên Console.

Một lỗi nhỏ, dễ sửa, nhưng nếu không để ý kỹ nội dung log, có thể khiến người mới bối rối hoặc — tệ hơn — tắt hẳn Alarm vì nghĩ nó "báo động vớ vẩn", trong khi vấn đề chỉ là cấu hình sai chiều so sánh.