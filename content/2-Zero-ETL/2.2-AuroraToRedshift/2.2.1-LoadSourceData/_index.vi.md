---
title : "Tải dữ liệu nguồn"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.2.1 </b> "
---

## Tóm Tắt:
- Tải cơ sở dữ liệu giao dịch của bạn trước khi tạo Zero-ETL Integration. Đồng bộ dữ liệu lần đầu được gọi là SEEDING (đồng bộ cấp lưu trữ).

- Kiểm tra khả năng lọc các bảng nguồn hoặc lược đồ bằng cách sử dụng DATA FILTERING trong quá trình tạo Zero-ETL Integration.

- Đối với Zero-ETL Integration hoạt động, trải nghiệm việc sao chép dữ liệu gần thời gian thực qua CDC (Change Data Capture). Tùy chọn, thử sử dụng HISTORY MODE.

## Kết nối với RDS MySQL, Tạo và Tải bảng cho SEEDING.

1. Để kết nối với phiên bản RDS MySQL của bạn
+ Điều hướng đến **EC2 console**
+ Chọn **EC2 instance** đã được cung cấp và nhấn vào **Connect**.

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. Trong trang **Connect**
+ Chọn **Session manager**
+ Chọn **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ Bạn sẽ được chuyển đến CLI của EC2 instance

3. Bạn cần lấy Endpoint và Port của RDS MySQL

+ Điều hướng đến **Aurora và RDS Console**
+ Chọn tính năng **Databases**
+ Chọn cơ sở dữ liệu với tiền tố tên **zero-etl-lab-sourcerdsmscluster-***
+ Chọn **Role Writer** dưới mặc định chọn
+ Sao chép **endpoint** của writer

![Create Stack](/images/2.Zero-ETLIntegration/57.png)

4. Quay lại CLI **EC2 instance**
+ Sử dụng lệnh sau để kết nối đến **Aurora MySQL** từ **EC2 instance**

`mysql -h rds_mysql_instance_endpoint -P 6612 -u awsuser -p `

+ Mật khẩu để đăng nhập

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/58.png)

5. Sau khi kết nối với **RDS for MySQL instance**
+ Thực thi các DDL dưới đây để tạo bảng cho bộ dữ liệu order-line.

```
create database seedingdemo;

use seedingdemo;

CREATE TABLE customer_info
(
customer_id BIGINT primary key,
job_title VARCHAR(500),
email_address VARCHAR(100),
full_name VARCHAR(200),
phone_number VARCHAR(20),
city VARCHAR(50),
state VARCHAR(50)
);
```

![Create Stack](/images/2.Zero-ETLIntegration/59.png)

6. Tải dữ liệu vào bảng đã tạo ở trên

```
LOAD DATA LOCAL INFILE '/zetldata/customer_info.csv' INTO TABLE customer_info FIELDS TERMINATED BY ',' IGNORE 1 ROWS;
```


![Create Stack](/images/2.Zero-ETLIntegration/60.png)

## Tạo và Tải bảng với khóa chính bị thiếu để minh họa DATA FILTERING.

1. Trên **RDS for MySQL instance** của bạn, chạy các DDL dưới đây để tạo bảng và lược đồ mà chúng ta sẽ **lọc ra** trong quá trình đồng bộ lần đầu (hoặc SEEDING). Lưu ý là chúng tôi đã cố tình loại bỏ khóa chính khỏi DDL để làm chúng trở thành ứng viên bị lọc ra.

```
create database filter_missingpk;

use filter_missingpk;

CREATE TABLE customer_info
(
customer_id BIGINT,
job_title VARCHAR(500),
email_address VARCHAR(100),
full_name VARCHAR(200),
phone_number VARCHAR(20),
city VARCHAR(50),
state VARCHAR(50)
);
```


![Create Stack](/images/2.Zero-ETLIntegration/61.png)

2. Tải dữ liệu vào bảng không có khóa chính đã tạo ở trên.

![Create Stack](/images/2.Zero-ETLIntegration/62.png)
