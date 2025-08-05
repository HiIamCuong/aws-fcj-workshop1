---
title : "Redshift Machine Learning"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 5.2 </b> "
---

Thông thường, khi chạy các mô hình Machine Learning, chúng ta cần kết hợp cả dữ liệu thời gian thực và dữ liệu lịch sử để nhận diện các xu hướng và phát hiện các giao dịch gian lận.

Trong phần lab này, chúng ta sẽ bắt đầu bằng cách tải một bảng dữ liệu lịch sử từ S3 bucket. So với dữ liệu streaming đã được ingest trước đó, dữ liệu lịch sử này có nhiều trường hơn.

Dữ liệu này sẽ bao gồm các thông tin như mức chi tiêu gần nhất của khách hàng và điểm rủi ro để xác định nguy cơ gian lận. Ngoài ra còn có một số biến khác như giao dịch vào cuối tuần hoặc giao dịch vào ban đêm. Những biến này là cốt lõi để nhận diện các xu hướng như đã đề cập trước đó.

## Tạo và tải dữ liệu lịch sử

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

4. Kết nối với kho dữ liệu Redshift Serverless bản preview  
+ Chọn **Other ways to connect**  
+ Chọn **Database user name and password**  
+ Nhập **User name**: `awsuser`  
+ Nhập **Password**: `Awsuser123`  
+ Chọn **Create connection**.

![Xác minh & Giám sát](/images/2.Zero-ETLIntegration/38.png)

5. Hãy tạo và tải dữ liệu vào bảng lịch sử giao dịch với lệnh SQL sau:
Vị trí bộ dữ liệu trên S3:

US East (N. Virginia) - vùng us-east-1 - s3://redshift-demos/ri2023/ant307/data/cust_payment_tx_history/

US West (Oregon) - vùng us-west-2 - s3://redshift-immersionday-labs/ri2023/ant307/data/cust_payment_tx_history/

```
CREATE TABLE cust_payment_tx_history
(
TRANSACTION_ID integer,
TX_DATETIME timestamp,
CUSTOMER_ID integer,
TERMINAL_ID integer,
TX_AMOUNT decimal(9,2),
TX_TIME_SECONDS integer,
TX_TIME_DAYS integer,
TX_FRAUD integer,
TX_FRAUD_SCENARIO integer,
TX_DURING_WEEKEND integer,
TX_DURING_NIGHT integer,
CUSTOMER_ID_NB_TX_1DAY_WINDOW decimal(9,2),
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW decimal(9,2),
CUSTOMER_ID_NB_TX_7DAY_WINDOW decimal(9,2),
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW decimal(9,2),
CUSTOMER_ID_NB_TX_30DAY_WINDOW decimal(9,2),
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW decimal(9,2),
TERMINAL_ID_NB_TX_1DAY_WINDOW decimal(9,2),
TERMINAL_ID_RISK_1DAY_WINDOW decimal(9,2),
TERMINAL_ID_NB_TX_7DAY_WINDOW decimal(9,2),
TERMINAL_ID_RISK_7DAY_WINDOW decimal(9,2),
TERMINAL_ID_NB_TX_30DAY_WINDOW decimal(9,2),
TERMINAL_ID_RISK_30DAY_WINDOW decimal(9,2)
);

COPY cust_payment_tx_history
FROM '<DÙNG ĐỊA CHỈ S3 NHƯ ĐÃ CHỈ ĐỊNH Ở TRÊN>'
IAM_ROLE default
PARQUET;
```


![ML](/images/6.ML/1.png)

![ML](/images/6.ML/2.png)

6. Bây giờ hãy kiểm tra xem có bao nhiêu giao dịch đã được tải vào:

```
SELECT
count(1)
FROM
cust_payment_tx_history;
```

![ML](/images/6.ML/3.png)

7. Sau đó, kiểm tra xu hướng giao dịch gian lận và không gian lận theo tháng:

```
SELECT
to_char(tx_datetime, 'YYYYMM') AS YearMonth,
sum(case when tx_fraud=1 then 1 else 0 end) AS fraud_tx,
sum(case when tx_fraud=0 then 1 else 0 end) AS non_fraud_tx,
count(*) AS total_tx
FROM cust_payment_tx_history
GROUP BY YearMonth;
```


![ML](/images/6.ML/4.png)

**Chúc mừng!** Bạn đã thiết lập xong bảng dữ liệu trong cụm Redshift của mình. Giờ đây chúng ta có thể bắt đầu xây dựng mô hình ML để dự đoán.
