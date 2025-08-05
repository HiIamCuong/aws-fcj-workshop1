---
title : "Thực thi dự đoán"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 5.2.3 </b> "
---

Bây giờ mô hình của chúng ta đã được huấn luyện và sẵn sàng sử dụng, chúng ta có thể sử dụng mô hình đã huấn luyện để chạy các dự đoán trên dữ liệu phát trực tuyến mà chúng ta đã nạp vào Redshift trước đó.

Chúng ta có thể áp dụng các phép biến đổi lên dữ liệu phát trực tuyến một cách rất dễ dàng bằng cách nhúng SQL vào bên trong các view. Chúng ta sẽ tạo một view trên dữ liệu của mình để tổng hợp dữ liệu phát trực tuyến ở cấp độ khách hàng. Sau đó, chúng ta sẽ tạo view thứ hai để tổng hợp dữ liệu ở cấp độ thiết bị đầu cuối và cuối cùng là view thứ ba, sẽ kết hợp dữ liệu giao dịch đến với dữ liệu tổng hợp của khách hàng và thiết bị đầu cuối và sẽ gọi hàm dự đoán tất cả trong một nơi.

## Tạo các view

Tạo view đầu tiên, tổng hợp dữ liệu phát trực tuyến ở cấp độ khách hàng:

```
CREATE VIEW public.customer_transformations
AS SELECT
customer_id,
CUSTOMER_ID_NB_TX_1DAY_WINDOW,
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW,
CUSTOMER_ID_NB_TX_7DAY_WINDOW,
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW,
CUSTOMER_ID_NB_TX_30DAY_WINDOW,
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW
FROM (
SELECT
customer_id,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) = cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end) AS CUSTOMER_ID_NB_TX_1DAY_WINDOW,
AVG(CASE WHEN cast(a.TX_DATETIME AS date) = cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end ) AS CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end) AS CUSTOMER_ID_NB_TX_7DAY_WINDOW,
AVG(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end ) AS CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -30 AND cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end) AS CUSTOMER_ID_NB_TX_30DAY_WINDOW,
AVG( CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -30 AND cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end ) AS CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW
FROM (
SELECT
CUSTOMER_ID,
TERMINAL_ID,
TX_AMOUNT,
cast(TX_DATETIME AS timestamp) TX_DATETIME
FROM CUST_PAYMENT_TX_STREAM -- lấy dữ liệu phát trực tuyến
WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
UNION
SELECT
CUSTOMER_ID,
TERMINAL_ID,
TX_AMOUNT,
cast(TX_DATETIME AS timestamp) TX_DATETIME
FROM cust_payment_tx_history -- lấy dữ liệu lịch sử
WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
) a
GROUP BY 1
);
```


![ML](/images/6.ML/10.png)

Tiếp theo, tạo view thứ hai, tổng hợp dữ liệu phát trực tuyến ở cấp thiết bị đầu cuối:

```
CREATE VIEW public.terminal_transformations
AS SELECT
TERMINAL_ID,
TERMINAL_ID_NB_TX_1DAY_WINDOW,
TERMINAL_ID_RISK_1DAY_WINDOW,
TERMINAL_ID_NB_TX_7DAY_WINDOW,
TERMINAL_ID_RISK_7DAY_WINDOW,
TERMINAL_ID_NB_TX_30DAY_WINDOW,
TERMINAL_ID_RISK_30DAY_WINDOW
FROM (
SELECT
TERMINAL_ID,
MAX(cast(a.TX_DATETIME AS date) ) maxdt,
MIN(cast(a.TX_DATETIME AS date) ) mindt,
MAX(tx_fraud) mxtf,
MIN(tx_fraud) mntf,
SUM(CASE WHEN tx_fraud =1 THEN 1 ELSE 0 end) sumtxfraud,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -7 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY1,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN tx_fraud ELSE 0 end) AS NB_TX_DELAY1 ,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -8 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY_WINDOW1,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -8 AND cast(getdate() AS date) THEN 1 ELSE 0 end) AS NB_TX_DELAY_WINDOW1,
NB_FRAUD_DELAY_WINDOW1-NB_FRAUD_DELAY1 AS NB_FRAUD_WINDOW1,
NB_TX_DELAY_WINDOW1-NB_TX_DELAY1 AS TERMINAL_ID_NB_TX_1DAY_WINDOW,
CASE WHEN TERMINAL_ID_NB_TX_1DAY_WINDOW = 0
THEN 0
ELSE NB_FRAUD_WINDOW1 / TERMINAL_ID_NB_TX_1DAY_WINDOW
end AS terminal_id_risk_1day_window ,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -7 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end ) AS NB_FRAUD_DELAY7,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN 1 ELSE 0 end) AS NB_TX_DELAY7,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -14 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY_WINDOW7,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -14 AND cast(getdate() AS date) THEN 1 ELSE 0 end) AS NB_TX_DELAY_WINDOW7,
NB_FRAUD_DELAY_WINDOW7-NB_FRAUD_DELAY7 AS NB_FRAUD_WINDOW7,
NB_TX_DELAY_WINDOW7-NB_TX_DELAY7 AS TERMINAL_ID_NB_TX_7DAY_WINDOW,
CASE WHEN TERMINAL_ID_NB_TX_7DAY_WINDOW = 0
THEN 0
ELSE NB_FRAUD_WINDOW7 / TERMINAL_ID_NB_TX_7DAY_WINDOW
END AS terminal_id_risk_7day_window,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date)-7 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY30,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date)-7 AND cast(getdate() AS date) THEN 1 ELSE 0 end) AS NB_TX_DELAY30,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date)-37 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY_WINDOW30,
SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN cast(getdate() AS date)-37 AND cast(getdate() AS date) THEN 1 ELSE 0 end) AS NB_TX_DELAY_WINDOW30,
NB_FRAUD_DELAY_WINDOW30-NB_FRAUD_DELAY30 AS NB_FRAUD_WINDOW30,
NB_TX_DELAY_WINDOW30-NB_TX_DELAY30 AS TERMINAL_ID_NB_TX_30DAY_WINDOW,
CASE WHEN TERMINAL_ID_NB_TX_30DAY_WINDOW = 0
THEN 0
ELSE NB_FRAUD_WINDOW30 / TERMINAL_ID_NB_TX_30DAY_WINDOW
END AS TERMINAL_ID_RISK_30DAY_WINDOW
FROM(
SELECT
TERMINAL_ID,
TX_AMOUNT,
cast(TX_DATETIME AS timestamp) TX_DATETIME,
0 AS TX_FRAUD
FROM cust_payment_tx_stream
WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
UNION ALL
SELECT
TERMINAL_ID,
TX_AMOUNT,
TX_DATETIME,
TX_FRAUD
FROM cust_payment_tx_history
WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
) a
GROUP BY 1
);
```

![ML](/images/6.ML/11.png)

Tạo view thứ ba, kết hợp dữ liệu giao dịch đến với dữ liệu tổng hợp khách hàng và thiết bị đầu cuối và gọi hàm dự đoán tại cùng một nơi:

```
CREATE VIEW public.cust_payment_tx_fraud_predictions
AS SELECT
A.APPROXIMATE_ARRIVAL_TIMESTAMP,
D.FULL_NAME,
D.EMAIL_ADDRESS,
D.PHONE_NUMBER,
A.TRANSACTION_ID,
A.TX_DATETIME,
A.CUSTOMER_ID,
A.TERMINAL_ID,
A.TX_AMOUNT ,
A.TX_TIME_SECONDS ,
A.TX_TIME_DAYS ,
fn_customer_cc_fd(
A.TX_AMOUNT ,
A.TX_DURING_WEEKEND,
A.TX_DURING_NIGHT,
C.CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
C.CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
C.CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
C.CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
C.CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
C.CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
T.TERMINAL_ID_NB_TX_1DAY_WINDOW ,
T.TERMINAL_ID_RISK_1DAY_WINDOW ,
T.TERMINAL_ID_NB_TX_7DAY_WINDOW ,
T.TERMINAL_ID_RISK_7DAY_WINDOW ,
T.TERMINAL_ID_NB_TX_30DAY_WINDOW ,
T.TERMINAL_ID_RISK_30DAY_WINDOW
) FRAUD_PREDICTION
FROM(
SELECT
APPROXIMATE_ARRIVAL_TIMESTAMP,
TRANSACTION_ID,
TX_DATETIME,
CUSTOMER_ID,
TERMINAL_ID,
TX_AMOUNT,
TX_TIME_SECONDS,
TX_TIME_DAYS,
CASE WHEN extract(dow from cast(TX_DATETIME as timestamp)) in (1,7) then 1 else 0 end as TX_DURING_WEEKEND,
CASE WHEN extract(hour from cast(TX_DATETIME as timestamp)) BETWEEN 00 and 06 then 1 else 0 end as TX_DURING_NIGHT
FROM public.cust_payment_tx_stream
) a
JOIN public.terminal_transformations t ON a.terminal_id = t.terminal_id
JOIN public.customer_transformations c ON a.customer_id = c.customer_id
JOIN rdsmysql_zeroetl.seedingdemo.customer_info d ON a.customer_id = d.customer_id
WITH NO SCHEMA BINDING;
```


![ML](/images/6.ML/12.png)

Chạy câu lệnh SELECT trên view:

```
SELECT *
FROM cust_payment_tx_fraud_predictions
WHERE FRAUD_PREDICTION = 1;
```

![ML](/images/6.ML/13.png)

Khi bạn chạy câu lệnh SELECT nhiều lần, các giao dịch thẻ tín dụng mới nhất sẽ được xử lý qua các phép biến đổi và dự đoán ML gần như theo thời gian thực.

Chúc mừng! Bạn đã thực hiện dự đoán thời gian thực bằng cách sử dụng phương pháp Redshift Zero-ETL.