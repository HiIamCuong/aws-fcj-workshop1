---
title : "Truy vấn Data Lake bằng Redshift Spectrum"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 4.1 </b> "
---

Trong phòng lab này, chúng tôi sẽ hướng dẫn bạn cách truy vấn dữ liệu trong Amazon S3 data lake bằng Amazon Redshift mà không cần tải hoặc di chuyển dữ liệu. Chúng tôi cũng sẽ trình bày cách bạn có thể tận dụng các view để kết hợp dữ liệu trong Redshift Managed storage với dữ liệu trong S3. Bạn có thể truy vấn dữ liệu có cấu trúc và bán cấu trúc từ các tệp trong Amazon S3 mà không cần sao chép hoặc di chuyển dữ liệu vào bảng Amazon Redshift. Để biết hướng dẫn mới nhất về các loại tệp có thể được truy vấn bằng Redshift Spectrum, vui lòng tham khảo [định dạng dữ liệu được hỗ trợ](https://docs.aws.amazon.com/redshift/latest/dg/c-spectrum-data-files.html#:~:text=Spectrum%20Regions.-,Data%20formats%20for%20Redshift%20Spectrum,-Redshift%20Spectrum%20supports).

## Mô tả trường hợp sử dụng
**Mục tiêu:** Rút ra thông tin chi tiết từ dữ liệu để tìm ảnh hưởng của việc rò rỉ dữ liệu thẻ tín dụng khách hàng bằng cách xác định các ngày trong tháng 1 năm 2023 có số giao dịch gian lận nhiều nhất.

**Mô tả tập dữ liệu:** Dữ liệu giao dịch thẻ tín dụng của khách hàng theo năm và tháng

**Vị trí S3 của tập dữ liệu:**

**Khu vực us-east-1 (N. Virginia)** - "s3://redshift-demos/ri2023/ant307/data/cust_payment_tx_history/" [us-east-1_s3_url](https://s3.console.aws.amazon.com/s3/buckets/redshift-demos?region=us-east-1&prefix=ri2023/ant307/data/cust_payment_tx_history/) 

**Khu vực us-west-2 (Oregon)** - "s3://redshift-immersionday-labs/ri2023/ant307/data/cust_payment_tx_history/" [us-west-2_s3_url](https://s3.console.aws.amazon.com/s3/buckets/redshift-immersionday-labs?region=us-west-2&prefix=ri2023/ant307/data/cust_payment_tx_history/) 

Dưới đây là cái nhìn tổng quan về các bước thực hiện trong phòng lab này.

## Hướng dẫn
1. Chạy Glue crawler để tạo Glue data catalog

Trong phòng lab này, chúng ta sẽ thực hiện các hoạt động sau:

+ Truy vấn dữ liệu lịch sử nằm trên S3 bằng cách tạo cơ sở dữ liệu ngoài cho Redshift Spectrum.

+ Khám phá dữ liệu lịch sử, có thể tổng hợp dữ liệu theo cách mới để thấy xu hướng theo thời gian, hoặc theo các chiều khác.

Lưu ý lược đồ phân vùng là Năm, Tháng. Đây là ảnh chụp màn hình: https://s3.console.aws.amazon.com/s3/buckets/redshift-demos?region=us-west-2&prefix=ri2023/ant307/data/cust_payment_tx_history/

![ProducingData](/images/4.FederatedQuery/1.png)

### Tạo external schema (và DB) cho Redshift Spectrum

Bạn có thể tạo bảng ngoài trong Amazon Redshift, AWS Glue, Amazon Athena hoặc metastore Apache Hive. Nếu bảng ngoài được định nghĩa trong AWS Glue, Athena hoặc Hive metastore, bạn cần tạo một external schema tham chiếu đến cơ sở dữ liệu ngoài. Sau đó, bạn có thể tham chiếu bảng ngoài trong câu lệnh SELECT bằng cách thêm tiền tố tên schema mà không cần tạo bảng trong Amazon Redshift.

Trong lab này, bạn sẽ sử dụng AWS Glue Crawler để tạo bảng ngoài `ant307_external.cust_payment_tx_history` được lưu trữ theo định dạng parquet tại vị trí s3://us-east-1.redshift-demos/ri2023/ant307/data/cust_payment_tx_history/

+ Truy cập trang **Glue Crawler**.

Khu vực us-east-1 - https://us-east-1.console.aws.amazon.com/glue/home 

Khu vực us-west-2 - https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2 

Trong lab này, crawler đã được tạo sẵn. Chọn crawler - CustPaymentTxHistory và nhấn Run.

![ProducingData](/images/4.FederatedQuery/2.png)

![ProducingData](/images/4.FederatedQuery/3.png)

+ Sau khi crawler chạy xong, bạn sẽ thấy bảng `cust_payment_tx_history` mới trong Glue Catalog

![ProducingData](/images/4.FederatedQuery/4.png)

2. Tạo external schema `ant307_external` trong Redshift và truy vấn từ bảng Glue catalog - `cust_payment_tx_history`

Truy cập Redshift console.
Khu vực us-east-1 - https://us-east-1.console.aws.amazon.com/redshiftv2/home?region=us-east-1#/serverless-setup 

Khu vực us-west-2 - https://us-west-2.console.aws.amazon.com/redshiftv2/home?region=us-west-2#/serverless-setup 

Nhấn vào mục **Serverless dashboard** ở menu bên trái. Chọn namespace đã tạo trước đó. Nhấn **Query data**.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

Tạo external schema **ant307_external** trỏ đến Glue Catalog Database **spectrumdb**.

```
CREATE external SCHEMA ant307_external
FROM data catalog DATABASE 'spectrumdb'
IAM_ROLE default
CREATE external DATABASE if not exists;
```

![ProducingData](/images/4.FederatedQuery/6.png)

### Tìm tháng có số lượng giao dịch gian lận cao nhất
Bạn có thể truy vấn bảng `cust_payment_tx_history` được định nghĩa trong Glue Catalog từ Redshift external schema. Trong tháng 1 năm 2023, có một ngày có số lượng giao dịch gian lận cao nhất. Bạn có thể tìm ra ngày đó?

```
SELECT TO_CHAR(TX_DATETIME, 'YYYY-MM-DD'),sum(tx_fraud)
FROM ant307_external.cust_payment_tx_history
WHERE d_year = 2023 and d_month = 01
GROUP BY 1
ORDER BY 2 desc;
```


![ProducingData](/images/4.FederatedQuery/7.png)

3. Tạo internal schema `ant307_das`

+ Tạo schema `ant307_das` cho các bảng sẽ được lưu trữ trong Redshift Managed Storage.

```
CREATE SCHEMA ant307_das;
```

![ProducingData](/images/4.FederatedQuery/8.png)

4. Chạy CTAS để tạo và nạp bảng Redshift `ant307_das.cust_payment_tx_202301` từ bảng ngoài

```
CREATE TABLE ant307_das.cust_payment_tx_202301 AS
SELECT *
FROM ant307_external.cust_payment_tx_history
WHERE d_year = 2023 AND d_month = 1;
```


![ProducingData](/images/4.FederatedQuery/9.png)

5. Xoá phân vùng 202301 khỏi bảng ngoài

Bây giờ chúng ta đã nạp toàn bộ dữ liệu tháng 1 năm 2023, ta có thể xoá phân vùng khỏi bảng Spectrum để tránh trùng lặp giữa bảng Redshift Managed Storage và bảng Spectrum.

```
ALTER TABLE ant307_external.cust_payment_tx_history DROP PARTITION(d_year=2023, d_month=1);
```


![ProducingData](/images/4.FederatedQuery/10.png)

6. Tạo view kết hợp `public.ant307_view_cust_payment_tx`

```
CREATE VIEW ant307_view_cust_payment_tx AS
SELECT * FROM ant307_das.cust_payment_tx_202301
UNION ALL
SELECT * FROM ant307_external.cust_payment_tx_history
WITH NO SCHEMA BINDING;
```


![ProducingData](/images/4.FederatedQuery/11.png)

### EXPLAIN hiển thị kế hoạch thực thi của một câu truy vấn mà không thực thi truy vấn

+ Lưu ý việc sử dụng các cột phân vùng trong các câu lệnh SELECT và WHERE. Những cột đó nằm ở đâu trong định nghĩa bảng Spectrum của bạn?

+ Lưu ý các bộ lọc được áp dụng ở mức phân vùng hoặc tệp trong tập dữ liệu Spectrum (so với tập dữ liệu trong Redshift Managed Storage).

+ Nếu bạn thực sự chạy truy vấn (không chỉ tạo kế hoạch EXPLAIN), thời gian chạy có khiến bạn bất ngờ không? Vì sao?

```
EXPLAIN
SELECT d_year, d_month, COUNT(*)
FROM ant307_view_cust_payment_tx
WHERE d_year = 2023 AND d_month IN (1,2) and terminal_id = 9999
GROUP BY 1,2
ORDER BY 1,2;
```

![ProducingData](/images/4.FederatedQuery/13.png)