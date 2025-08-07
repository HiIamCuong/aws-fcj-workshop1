---
title : "Phân tích tất cả dữ liệu của bạn"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

{{% notice info %}}  
Đảm bảo rằng [Tích hợp Zero-ETL](2-Zero-ETL) của bạn đang hoạt động và xác minh rằng dữ liệu từ cụm Aurora MySQL đã được đồng bộ hóa với Amazon Redshift.  
{{% /notice %}}

## Tổng hợp lại tất cả

1. Để tạo cơ sở dữ liệu của bạn, hãy hoàn thành các bước sau:
+ Trên bảng điều khiển Redshift Serverless, điều hướng đến namespace bắt đầu bằng **zero-etl-destination-***.
+ Chọn **Query data** trên bảng điều khiển namespace Redshift Serverless để mở [Query Editor v2](https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html). Tiến hành *configure* nếu được yêu cầu cấp quyền.

![Xác minh & Giám sát](/images/2.Zero-ETLIntegration/34.png)

2. Trong trang cấu hình  
+ Chọn configure account

![Xác minh & Giám sát](/images/2.Zero-ETLIntegration/36.png)

3. Trong Redshift Query Editor V2  
+ Nhấp vào data warehouse có tiền tố tên **zero-etl-destination-***

![Xác minh & Giám sát](/images/2.Zero-ETLIntegration/37.png)

4. Kết nối với kho dữ liệu Redshift Serverless bản xem trước  
+ Chọn **Other ways to connect**  
+ Chọn **Database user name and password**  
+ Điền **User name**: `awsuser`  
+ Điền **Password**: `Awsuser123`  
+ Chọn **Create connection**.

![Xác minh & Giám sát](/images/2.Zero-ETLIntegration/38.png)

5. Hãy chạy một truy vấn phân tích để phân tích tất cả các tập dữ liệu nguồn ngay lập tức. Những ý tưởng tuyệt vời không nên chờ đợi. Hãy xem cách ExampleCorp sẽ triển khai chiến dịch Pop-Up ngay lập tức bằng cách phân tích tất cả dữ liệu của mình.

```
---- ExampleCorp muốn triển khai một chiến dịch khuyến mãi có mục tiêu cho 100 khách hàng ưu tiên cao nhất với mức chi tiêu lớn nhất.

SELECT c.full_name,
c.email_address,
o.o_orderpriority,
Sum(h.tx_amount) cust_past_spend,
Sum(p.tx_amount) cust_curr_spend
FROM auroramysql_zeroetl.seedingdemo.orders o ------ Nguồn tích hợp Zero-ETL: Aurora MySQL
-- INNER JOIN postgres.customer_info c ------ Nguồn truy vấn liên kết: Aurora Postgres (Dữ liệu này nằm trong 3 hệ thống; chọn một)
-- INNER JOIN rdsmysql_zeroetl.seedingdemo.customer_info c ------ Nguồn tích hợp Zero-ETL: RDS for MySQL
INNER JOIN aurorapostgres_zeroetl.public.customer_info c ------ Nguồn tích hợp Zero-ETL: Aurora PostgreSQL
ON o.o_custkey = c.customer_id
INNER JOIN (SELECT customer_id,
Sum(tx_amount) tx_amount
FROM ant307_external.cust_payment_tx_history
GROUP BY 1) h ------ Nguồn truy vấn liên kết: Amazon S3
ON o.o_custkey = h.customer_id
INNER JOIN cust_payment_tx_stream p ------ Nguồn dữ liệu streaming: Kinesis data streams
ON p.customer_id = c.customer_id
WHERE o.o_orderpriority = '1-URGENT'
GROUP BY 1,
2,
3
ORDER BY 4 DESC,
5 DESC
LIMIT 100;
```


![Phân tích](/images/5.Analyze/1.png)

**Tùy chọn khác** - Sử dụng tích hợp Zero-ETL từ RDS for MySQL với Amazon Redshift.

![Phân tích](/images/5.Analyze/2.png)

**Chúc mừng!** Bạn đã hoàn tất phương pháp Zero-ETL cho một chiến dịch phân tích mẫu. Tiếp tục khám phá dữ liệu nhé!