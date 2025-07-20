---
title : "Tích hợp Zero-ETL Aurora MySQL sang Redshift"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

Được công bố tại sự kiện AWS [re:Invent 2023](https://youtu.be/PMfn9_nTDbM?t=7418) và hiện đã khả dụng rộng rãi ([GA](https://aws.amazon.com/about-aws/whats-new/2024/09/amazon-rds-mysql-zero-etl-integration-redshift-generally-available/)) tại nhiều khu vực cho RDS for MySQL phiên bản 8.0.

## Lợi ích

Với tính năng **[Tích hợp Zero-ETL](1-Zero-ETL/)** giữa **RDS for MySQL** và **Amazon Redshift**, bạn có thể kết hợp dữ liệu giao dịch từ **RDS for MySQL** với khả năng phân tích mạnh mẽ của **Amazon Redshift**. Tính năng này giúp giảm thiểu công việc xây dựng và quản lý các pipeline ETL tùy chỉnh giữa **RDS for MySQL** và **Amazon Redshift**. Các kỹ sư dữ liệu hiện có thể sao chép dữ liệu từ nhiều cụm cơ sở dữ liệu **RDS for MySQL** vào cùng một hoặc một phiên bản **Amazon Redshift** mới để rút ra các hiểu biết tổng thể trên nhiều ứng dụng hoặc phân vùng khác nhau. Các cập nhật trong **RDS for MySQL** được truyền tự động và liên tục sang **Amazon Redshift**, đảm bảo các kỹ sư dữ liệu luôn có thông tin mới nhất gần như thời gian thực. Ngoài ra, toàn bộ hệ thống có thể là không máy chủ (**serverless**) và có khả năng tự động mở rộng hoặc thu nhỏ tùy theo khối lượng dữ liệu, giúp bạn không phải quản lý hạ tầng.

## Tiến trình

Tính năng tích hợp này sao chép dữ liệu từ cơ sở dữ liệu nguồn vào kho dữ liệu đích. Dữ liệu sẽ có sẵn trong **Amazon Redshift** chỉ sau vài giây, cho phép người dùng tận dụng các tính năng phân tích của **Amazon Redshift** và các khả năng như chia sẻ dữ liệu, tối ưu hóa khối lượng công việc tự động, mở rộng đồng thời, học máy và nhiều tính năng khác. Bạn có thể xử lý giao dịch theo thời gian thực trên dữ liệu trong **RDS for MySQL** trong khi đồng thời sử dụng **Amazon Redshift** cho các khối lượng công việc phân tích như báo cáo và bảng điều khiển.

Tính năng tích hợp này sẽ giám sát tình trạng của pipeline dữ liệu và tự động khắc phục sự cố nếu có thể. Bạn có thể tạo tích hợp từ nhiều cụm **RDS for MySQL** vào một **namespace Amazon Redshift** duy nhất, cho phép rút ra các phân tích từ nhiều ứng dụng.

## Chi phí

{{% notice notice about cost %}}
Khi bạn tạo tích hợp **Zero-ETL** giữa **RDS for MySQL** và **Amazon Redshift**, bạn vẫn thanh toán theo mức giá hiện tại của **RDS MySQL** và **Amazon Redshift** (bao gồm chi phí truyền dữ liệu). Tính năng tích hợp này không tính thêm phí.
{{% /notice %}}

Bạn sẽ thanh toán cho tài nguyên **Amazon RDS for MySQL** và **Amazon Redshift** hiện có được sử dụng để tạo và xử lý dữ liệu thay đổi trong quá trình tích hợp **Zero-ETL**. Để biết thêm thông tin, hãy xem giá của [Amazon RDS for MySQL](https://aws.amazon.com/rds/mysql/pricing/) và [Amazon Redshift](https://aws.amazon.com/redshift/pricing/).
