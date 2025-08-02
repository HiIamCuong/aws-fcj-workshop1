---
title : "Tích hợp Zero-ETL Aurora MySQL với Redshift"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---

Được công bố tại [AWS re:Invent 2023](https://youtu.be/PMfn9_nTDbM?t=7418) và hiện đã có mặt chính thức ([GA](https://aws.amazon.com/about-aws/whats-new/2024/09/amazon-rds-mysql-zero-etl-integration-redshift-generally-available/)) tại nhiều vùng cho **RDS for MySQL phiên bản 8.0**.

## Lợi ích

Với tính năng **[Zero-ETL Integration](1-Zero-ETL/)** giữa **RDS for MySQL** và **Amazon Redshift**, bạn có thể kết hợp dữ liệu giao dịch từ **RDS for MySQL** với khả năng phân tích mạnh mẽ của **Amazon Redshift**. Tính năng này giúp giảm thiểu công việc xây dựng và quản lý các pipeline ETL tùy chỉnh giữa hai dịch vụ này.

Các kỹ sư dữ liệu có thể sao chép dữ liệu từ nhiều cụm cơ sở dữ liệu **RDS for MySQL** vào cùng một cụm **Amazon Redshift** mới hoặc hiện có để có được cái nhìn tổng quan về nhiều ứng dụng hoặc phân vùng. Các cập nhật trong **RDS for MySQL** sẽ được tự động và liên tục đồng bộ đến **Amazon Redshift**, đảm bảo rằng dữ liệu luôn được cập nhật gần thời gian thực.

Hệ thống có thể hoạt động theo chế độ **serverless** và có khả năng **tự động co giãn** theo khối lượng dữ liệu, giúp bạn không cần phải quản lý hạ tầng phức tạp.

## Quy trình

Tích hợp này sao chép dữ liệu từ cơ sở dữ liệu nguồn sang kho dữ liệu đích. Dữ liệu sẽ có sẵn trong **Amazon Redshift** chỉ sau vài giây, cho phép người dùng tận dụng các tính năng phân tích mạnh mẽ như:

- Chia sẻ dữ liệu
- Tối ưu hóa tải công việc tự động
- Mở rộng đồng thời (concurrent scaling)
- Học máy (Machine Learning)
- Và nhiều tính năng khác

Bạn có thể xử lý giao dịch thời gian thực trên dữ liệu trong **RDS for MySQL**, đồng thời sử dụng **Amazon Redshift** để phục vụ các nhu cầu phân tích như báo cáo và bảng điều khiển (dashboard).

Tích hợp này theo dõi trạng thái của pipeline dữ liệu và sẽ tự động khắc phục sự cố khi có thể. Bạn có thể tạo tích hợp từ nhiều cụm **RDS for MySQL** vào một **namespace Amazon Redshift duy nhất**, từ đó phân tích tổng hợp trên nhiều ứng dụng khác nhau.

## Chi phí

{{% notice info %}}
Khi bạn tạo một tích hợp **Zero-ETL** giữa **RDS for MySQL** và **Amazon Redshift**, bạn vẫn sẽ thanh toán theo bảng giá hiện tại của **RDS for MySQL** và **Amazon Redshift** (bao gồm cả chi phí truyền dữ liệu). Tính năng tích hợp này **không tính thêm phí**.
{{% /notice %}}

Bạn sẽ bị tính phí cho các tài nguyên hiện có của **Amazon RDS for MySQL** và **Amazon Redshift** được sử dụng để tạo và xử lý các thay đổi dữ liệu trong quá trình tích hợp **Zero-ETL**.

Để biết thêm chi tiết, tham khảo:

- [Giá dịch vụ Amazon RDS for MySQL](https://aws.amazon.com/rds/mysql/pricing/)
- [Giá dịch vụ Amazon Redshift](https://aws.amazon.com/redshift/pricing/)