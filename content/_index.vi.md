---
title : "Session Management"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Kết nối và phân tích tất cả dữ liệu của bạn bằng phương pháp zero-ETL

### Tổng quan

 Trong bài lab này ta sẽ sử dụng chức năng zero-ETL của [Amazon Redshift](https://aws.amazon.com/redshift/) để có thể tổng hợp dữ liệu từ nhiều cụm cơ sở dữ liệu nguồn của [Amazon Aurora](https://aws.amazon.com/rds/aurora/) vào các kho dữ liệu của Redshift. Có thể cập nhật lại dữ liệu [Amazon Redshift](https://aws.amazon.com/redshift/) theo các cụm [Amazon Aurora](https://aws.amazon.com/rds/aurora/) gần như theo thời gian thực.

Sau đó, chúng ta sẽ sử dụng một số tính năng của [Amazon Redshift](https://aws.amazon.com/redshift/) để hỗ trợ phân tích toàn bộ dữ liệu và đưa ra quyết định cho doanh nghiệp của bạn, bao gồm:

- [**Redshift Streaming Ingestion**](3-RedshiftStreamingIngestion/) – Nhập dữ liệu luồng vào Redshift  
- [**Federated Query**](4-FederatedQueries) – Truy vấn dữ liệu từ nhiều nguồn khác nhau  
- [**Amazon Redshift ML**](5-Analytics&ML) – Tích hợp học máy để dự đoán và phân tích nâng cao  

![Architecture](/images/architecture.png)

### Nội dung
 1. [Chuẩn bị](1-Preparation/)
 2. [Zero-ETL](2-Zero-ETL/)
 3. [Redshift Streaming Ingestion](3-RedshiftStreamingIngestion/)
 4. [Truy vấn liên kết](4-FederatedQueries)
 5. [Phân tích & Học máy](5-Analytics&ML)
 6. [Dọn dẹp tài nguyên](6-cleanup/)
