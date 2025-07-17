---
title : "Session Management"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Kết nối và phân tích tất cả dữ liệu của bạn bằng phương pháp zero-ETL

### Tổng quan

 Trong bài lab này ta sẽ sử dụng chức năng zero-ETL của [Amazon Redshift](https://aws.amazon.com/redshift/) để có thể tổng hợp dữ liệu từ nhiều cụm cơ sở dữ liệu nguồn của [Amazon Aurora](https://aws.amazon.com/rds/aurora/) vào các kho dữ liệu của Redshift. Có thể cập nhật lại dữ liệu [Amazon Redshift](https://aws.amazon.com/redshift/) theo các cụm [Amazon Aurora](https://aws.amazon.com/rds/aurora/) gần như theo thời gian thực.


### Nội dung

 1. [Giới thiệu](1-introduce/)
 2. [Các bước chuẩn bị](2-Prerequiste/)
 3. [Tạo kết nối đến máy chủ EC2](3-Accessibilitytoinstance/)
 4. [Quản lý session logs](4-s3log/)
 5. [Port Forwarding](5-Portfwd/)
 6. [Dọn dẹp tài nguyên](6-cleanup/)
