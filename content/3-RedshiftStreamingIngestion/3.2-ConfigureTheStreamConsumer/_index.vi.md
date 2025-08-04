---
title : "Cấu hình consumer của luồng"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

## Tạo một schema external trong Redshift

1. Để tạo cơ sở dữ liệu, hãy hoàn thành các bước sau:
+ Trên bảng điều khiển Redshift Serverless, điều hướng đến namespace bắt đầu bằng **zero-etl-destination-***.
+ Chọn **Query data** trên giao diện namespace Redshift Serverless để mở [Query Editor v2](https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html). Nhấn *Configure* nếu được yêu cầu cấp quyền.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

2. Trên trang cấu hình:
+ Chọn **Configure account**

![Validations & Monitoring](/images/2.Zero-ETLIntegration/36.png)

3. Trong Redshift Query Editor V2:
+ Nhấn vào data warehouse có tên bắt đầu bằng **zero-etl-destination-***

![Validations & Monitoring](/images/2.Zero-ETLIntegration/37.png)

4. Kết nối với Redshift Serverless preview data warehouse:
+ Chọn **Other ways to connect**
+ Chọn **Database user name and password**
+ Nhập **User name**: `awsuser`
+ Nhập **Password**: `Awsuser123`
+ Chọn **Create connection**

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

5. Trong Redshift Query Editor, chạy lệnh SQL sau:

```
CREATE EXTERNAL SCHEMA custpaytxn
FROM KINESIS
IAM_ROLE default;
```


![ProducingData](/images/3.RedshiftStreamingIngestion/7.png)

Lệnh này sẽ tạo một **external schema** có tên là **custpaytxn**, schema này sẽ được dùng để ánh xạ tới Kinesis Data Stream.

Bạn có thể xác minh schema đã tạo thành công bằng cách chạy truy vấn sau:

```
select * from SVV_EXTERNAL_SCHEMAS;
```


![ProducingData](/images/3.RedshiftStreamingIngestion/8.png)

6. Bây giờ bạn có thể tạo một **materialized view** để tiêu thụ dữ liệu từ Kinesis. Bạn sẽ sử dụng hàm JSON trong Redshift để phân tích dữ liệu thành từng cột riêng biệt.

```
CREATE MATERIALIZED VIEW cust_payment_tx_stream
AUTO REFRESH YES
AS
SELECT approximate_arrival_timestamp ,
partition_key,
shard_id,
sequence_number,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TRANSACTION_ID')::bigint as TRANSACTION_ID,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_DATETIME')::character(50) as TX_DATETIME,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'CUSTOMER_ID')::int as CUSTOMER_ID,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TERMINAL_ID')::int as TERMINAL_ID,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_AMOUNT')::decimal(18,2) as TX_AMOUNT,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_TIME_SECONDS')::int as TX_TIME_SECONDS,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_TIME_DAYS')::int as TX_TIME_DAYS
FROM custpaytxn."cust-payment-txn-stream"
Where is_utf8(kinesis_data) AND can_json_parse(kinesis_data);
```


![ProducingData](/images/3.RedshiftStreamingIngestion/11.png)

Bạn có thể thấy rằng trong mã SQL trên, chúng ta đã sử dụng **AUTO REFRESH YES**, điều này cho phép tự động làm mới materialized view nhận dữ liệu streaming, giúp tiết kiệm thời gian vì không cần thiết lập pipeline cập nhật thủ công.

7. Xác nhận việc làm mới dữ liệu mỗi phút từ Kinesis vào materialized view trong Redshift bằng cách chạy lệnh SQL sau lặp lại mỗi phút:

```
Select count(*) from cust_payment_tx_stream;
```


![ProducingData](/images/3.RedshiftStreamingIngestion/12.png)

8. Bây giờ hãy truy vấn materialized view để xem dữ liệu mẫu:

```
Select * from cust_payment_tx_stream limit 10;
```


![ProducingData](/images/3.RedshiftStreamingIngestion/13.png)

9. Hãy kiểm tra xem có bao nhiêu bản ghi hiện có trong materialized view:

```
Select count(*) as stream_rec_count from cust_payment_tx_stream;
```

![ProducingData](/images/3.RedshiftStreamingIngestion/14.png)

{{% notice info %}}
Chúc mừng! Bạn đã thiết lập thành công quá trình ingest dữ liệu streaming từ Kinesis Data Stream vào Redshift.
{{% /notice %}}