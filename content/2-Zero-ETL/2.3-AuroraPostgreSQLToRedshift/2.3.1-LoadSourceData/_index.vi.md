---
title : "Tải dữ liệu nguồn"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.3.1 </b> "
---

## Dàn bài:
- Tải cơ sở dữ liệu giao dịch của bạn trước khi tạo Zero-ETL Integration. Quá trình đồng bộ lần đầu dữ liệu được gọi là SEEDING (đồng bộ cấp lưu trữ).

- Kiểm tra khả năng lọc các bảng nguồn hoặc schema sử dụng DATA FILTERING trong quá trình tạo Zero-ETL integration.

- Đối với một Zero-ETL integration hoạt động, trải nghiệm gần như đồng bộ dữ liệu thay đổi theo thời gian thực (CDC - Change Data Capture). Tùy chọn thử sử dụng HISTORY MODE.

## Kết nối với Aurora PostgreSQL, xác minh bảng SEEDING có sẵn.

Xem dữ liệu trên bảng customer_info trong cơ sở dữ liệu Aurora PostgreSQL.

1. Trong **Bảng điều khiển Aurora và RDS**
+ Điều hướng đến **Aurora PostgreSQL [Query Editor](https://console.aws.amazon.com/rds/home#query-editor:)**
+ Chọn cụm có tên **aurorapg-***.
+ Đối với tên người dùng cơ sở dữ liệu, chọn **Connect with a Secrets Manager ARN**.
+ Điền giá trị ARN (Cách lấy được giải thích ở bước 2). Nhập tên cơ sở dữ liệu là `postgres`.

![Create Stack](/images/2.Zero-ETLIntegration/112.png)

2. Kết nối với cơ sở dữ liệu Aurora PostgreSQL bằng ARN của Secrets Manager.
+ Bạn có thể lấy ARN từ tab **Outputs** trong [CloudFormation](https://console.aws.amazon.com/cloudformation/home),

![Create Stack](/images/2.Zero-ETLIntegration/113.png)

3. Sao chép câu lệnh sau vào Editor và chạy câu lệnh SQL.

```
select * from public.customer_info;
```


![Create Stack](/images/2.Zero-ETLIntegration/114.png)

![Create Stack](/images/2.Zero-ETLIntegration/115.png)

{{% notice info %}}
Việc tạo dữ liệu nguồn sẽ được sao chép qua Zero-ETL integration đã hoàn thành. Trong phần tiếp theo, chúng ta sẽ tạo ra một số dữ liệu không hữu ích mà chúng ta sẽ lọc trong quá trình tạo Zero-ETL integration. Ví dụ điển hình của bộ dữ liệu như vậy là bảng giao dịch thiếu khóa chính.
{{% /notice %}}

## Tạo và Tải các bảng thiếu khóa chính để minh họa DATA FILTERING.

1. Trong **Aurora PostgreSQL [Query Editor](https://console.aws.amazon.com/rds/home#query-editor:)**
+ Chạy các câu lệnh DDL dưới đây để tạo các bảng và schema mà chúng ta sẽ **lọc ra** trong quá trình đồng bộ lần đầu (hoặc SEEDING). Lưu ý rằng chúng ta đã cố ý loại bỏ khóa chính khỏi DDL để làm cho các bảng này trở thành ứng viên để lọc.

```
CREATE TABLE public.customer_info_missing_pk
(
customer_id BIGINT,
job_title VARCHAR(500),
email_address VARCHAR(100),
full_name VARCHAR(200),
phone_number VARCHAR(20),
city VARCHAR(50),
state VARCHAR(50)
);

insert into public.customer_info_missing_pk values (1, 'Dummy job title','dummyemail@example.com','Random Name','123-456-7890','Some City','Some State');
```

![Create Stack](/images/2.Zero-ETLIntegration/116.png)