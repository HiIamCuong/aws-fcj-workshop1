---
title : "Tích hợp zero-ETL giữa Aurora PostgreSQL và Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 2.3 </b> "
---

Được công bố tại sự kiện [AWS re:Invent 2023](https://youtu.be/PMfn9_nTDbM?t=7418) và hiện đã **chính thức ra mắt (GA)** tại [nhiều khu vực](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/zero-etl.html#zero-etl.regions), tính năng này hỗ trợ [Aurora PostgreSQL phiên bản 16](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html#Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL-Postgres).

## Lợi ích

Với [tích hợp Zero-ETL giữa Aurora PostgreSQL và Amazon Redshift](https://aws.amazon.com/rds/aurora/zero-etl/):

- Kết hợp dữ liệu giao dịch từ Aurora PostgreSQL với khả năng phân tích mạnh mẽ của Amazon Redshift.
- Không cần xây dựng và bảo trì pipeline ETL tùy chỉnh giữa Aurora và Redshift.
- Cho phép sao chép dữ liệu từ nhiều cụm Aurora PostgreSQL sang cùng một cụm Amazon Redshift để có cái nhìn toàn diện.
- Dữ liệu được đồng bộ hóa liên tục theo thời gian gần thực, đảm bảo luôn có thông tin mới nhất để phân tích.
- Hệ thống có thể hoạt động serverless và tự động co giãn theo khối lượng dữ liệu, giảm nhu cầu quản lý hạ tầng.

## Quy trình hoạt động

- Dữ liệu được sao chép từ cơ sở dữ liệu nguồn (Aurora PostgreSQL) sang kho dữ liệu Redshift.
- Dữ liệu khả dụng trong vài giây, hỗ trợ các tính năng phân tích của Redshift như:
  - Data sharing
  - Tối ưu hóa tải công việc tự động
  - Mở rộng đồng thời
  - Học máy, v.v.
- Hỗ trợ xử lý giao dịch trong Aurora PostgreSQL và phân tích đồng thời trong Redshift.
- Hệ thống tự động theo dõi pipeline và phục hồi khi có lỗi.
- Hỗ trợ tích hợp từ nhiều cụm Aurora PostgreSQL vào một namespace Amazon Redshift duy nhất.

![Architecture](/images/2.Zero-ETLIntegration/150.png)

## Giá thành

{{% notice info %}}
Khi sử dụng tích hợp Zero-ETL, bạn **chỉ cần thanh toán** cho Aurora PostgreSQL và Amazon Redshift theo bảng giá hiện tại (bao gồm cả chi phí truyền dữ liệu). **Không có phụ phí** cho việc sử dụng tính năng tích hợp Zero-ETL.
{{% /notice %}}

Bạn sẽ trả chi phí cho:
- Tài nguyên Aurora PostgreSQL để tạo và phát hiện dữ liệu thay đổi.
- Tài nguyên Amazon Redshift để xử lý và lưu trữ dữ liệu.

Tham khảo:
- [Giá Aurora PostgreSQL](https://aws.amazon.com/rds/aurora/pricing/)
- [Giá Amazon Redshift](https://aws.amazon.com/redshift/pricing/)