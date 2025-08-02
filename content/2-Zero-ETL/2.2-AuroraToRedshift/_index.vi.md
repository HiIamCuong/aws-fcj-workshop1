---
title : "Tích hợp Zero-ETL Amazon Aurora với Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.2 </b> "
---

Được công bố tại [AWS re:Invent 2022](https://youtu.be/Xus8C2s5K9A?t=2212) và hiện nay đã có sẵn ([GA](https://aws.amazon.com/about-aws/whats-new/2023/11/aws-general-availability-amazon-aurora-mysql-zero-etl-integration-redshift/)) tại [nhiều region](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html) cho [Amazon Aurora MySQL-Compatible Edition 3](https://aws.amazon.com/rds/aurora/mysql-features/) (phiên bản 3.05.2 trở lên, tương thích với MySQL 8.0.32 hoặc cao hơn — hãy kiểm tra [các phiên bản được hỗ trợ mới nhất](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html)).

## Lợi ích

Thông qua [Aurora Zero-ETL Integration với Amazon Redshift](2-Zero-ETL/), bạn có thể kết hợp dữ liệu giao dịch từ Aurora với khả năng phân tích mạnh mẽ của Amazon Redshift. Điều này giúp giảm thiểu công việc xây dựng và quản lý các pipeline ETL tùy chỉnh giữa Aurora và Redshift.

Các kỹ sư dữ liệu có thể sao chép dữ liệu từ nhiều cụm cơ sở dữ liệu Aurora vào cùng một hoặc một cụm Amazon Redshift mới để có cái nhìn tổng quan trên nhiều ứng dụng hoặc phân vùng. Các cập nhật trong Aurora sẽ được tự động và liên tục đồng bộ sang Redshift, mang lại dữ liệu gần thời gian thực.

Hệ thống có thể chạy ở chế độ serverless và tự động co giãn theo khối lượng công việc, giúp bạn không cần quản lý hạ tầng.

## Tiến trình hoạt động

Aurora Zero-ETL sẽ sao chép dữ liệu từ cơ sở dữ liệu nguồn sang kho dữ liệu đích (Amazon Redshift). Dữ liệu sẽ có sẵn trong Redshift chỉ sau vài giây, cho phép bạn sử dụng các tính năng như:

- Chia sẻ dữ liệu
- Tối ưu hóa tải công việc tự động
- Mở rộng đồng thời
- Học máy (Machine Learning)

Bạn vẫn có thể thực hiện các giao dịch thời gian thực trên Aurora trong khi đồng thời khai thác dữ liệu phân tích trong Redshift cho báo cáo và dashboard.

Tích hợp này theo dõi tình trạng pipeline dữ liệu và cố gắng phục hồi khi có lỗi. Bạn có thể tích hợp từ nhiều cụm Aurora DB vào một không gian tên Redshift duy nhất để truy xuất thông tin từ nhiều ứng dụng.

## Chi phí

{{% notice info %}}
Khi bạn tạo tích hợp Aurora Zero-ETL với Amazon Redshift, bạn **vẫn thanh toán cho Aurora và Redshift theo giá hiện hành** (bao gồm chi phí chuyển dữ liệu). Việc sử dụng tính năng tích hợp Zero-ETL **không phát sinh thêm chi phí**.
{{% /notice %}}

Bạn sẽ chi trả cho các tài nguyên sử dụng trong quá trình tích hợp, bao gồm:

- I/O và dung lượng lưu trữ bổ sung do bật ghi log nhị phân nâng cao
- Chi phí xuất snapshot để tải dữ liệu ban đầu vào Redshift
- Dung lượng lưu trữ bổ sung trong Redshift cho dữ liệu được sao chép
- Chuyển dữ liệu giữa các Availability Zone (AZ)

**Bạn sẽ không bị tính phí** cho việc sao chép dữ liệu thay đổi liên tục.

### Tham khảo thêm:

- [Giá Amazon Aurora](https://aws.amazon.com/rds/aurora/pricing/)
- [Giá Amazon RDS for MySQL](https://aws.amazon.com/rds/mysql/pricing/)
- [Giá Amazon Redshift](https://aws.amazon.com/redshift/pricing/)