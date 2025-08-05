---
title : "Xây dựng mô hình ML"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 5.2.2 </b> "
---

**Amazon Redshift ML** cho phép bạn huấn luyện mô hình chỉ với một lệnh SQL **CREATE MODEL** duy nhất. Câu lệnh **CREATE MODEL** sẽ tạo ra một mô hình mà Amazon Redshift có thể sử dụng để tạo ra các dự đoán dựa trên mô hình thông qua các cú pháp SQL quen thuộc.

**Amazon Redshift ML** hỗ trợ cả nhà phân tích dữ liệu và nhà khoa học dữ liệu trong việc sử dụng Machine Learning. Nó cũng cho phép các chuyên gia Machine Learning sử dụng kiến thức của họ để điều chỉnh câu lệnh **CREATE MODEL** nhằm chỉ định các yếu tố cụ thể. Bằng cách đó, bạn có thể tăng tốc quá trình tìm kiếm mô hình tối ưu, cải thiện độ chính xác, hoặc cả hai.

**CREATE MODEL** sử dụng **Amazon SageMaker Autopilot** phía sau để tự động thực hiện các bước chuẩn bị dữ liệu, tạo đặc trưng, chọn mô hình và huấn luyện — mà không cần tạo pipeline tùy chỉnh trong chính SageMaker.

Trong lab này, chúng ta sẽ sử dụng **CREATE MODEL** để huấn luyện và kiểm thử một mô hình trên bảng giao dịch thẻ lịch sử. Bảng này có dữ liệu trong vòng 6 tháng để sử dụng cho việc huấn luyện.

Mô hình sẽ sử dụng các trường sau làm đầu vào:

```
TX_DURING_WEEKEND ,
TX_AMOUNT,
TX_DURING_NIGHT ,
CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
TERMINAL_ID_NB_TX_1DAY_WINDOW ,
TERMINAL_ID_RISK_1DAY_WINDOW ,
TERMINAL_ID_NB_TX_7DAY_WINDOW ,
TERMINAL_ID_RISK_7DAY_WINDOW ,
TERMINAL_ID_NB_TX_30DAY_WINDOW ,
TERMINAL_ID_RISK_30DAY_WINDOW
```

Trường đầu ra là **tx_fraud**.

Chúng ta sẽ chia dữ liệu thành hai tập: huấn luyện và kiểm thử. Giao dịch từ **2022-04-01** đến **2022-07-31** sẽ dùng làm tập huấn luyện. Giao dịch từ **2022-08-01** đến **2022-09-30** sẽ dùng cho tập kiểm thử.

Hãy tạo mô hình ML bằng câu lệnh quen thuộc [SQL CREATE MODEL](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_MODEL.html). Phương pháp dưới đây sử dụng [Amazon SageMaker Autopilot](https://aws.amazon.com/sagemaker/autopilot/) để thực hiện tất cả các bước một cách tự động.

Thực thi đoạn mã sau (thay thế tên S3 bucket bằng bucket của bạn bắt đầu với tiền tố **zero-etl-approaches-file-store-***):

```
CREATE MODEL cust_cc_txn_fd
FROM (
SELECT
TX_AMOUNT ,
TX_FRAUD ,
TX_DURING_WEEKEND ,
TX_DURING_NIGHT ,
CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
TERMINAL_ID_NB_TX_1DAY_WINDOW ,
TERMINAL_ID_RISK_1DAY_WINDOW ,
TERMINAL_ID_NB_TX_7DAY_WINDOW ,
TERMINAL_ID_RISK_7DAY_WINDOW ,
TERMINAL_ID_NB_TX_30DAY_WINDOW ,
TERMINAL_ID_RISK_30DAY_WINDOW
FROM cust_payment_tx_history
WHERE cast(tx_datetime as date) BETWEEN '2022-06-01' AND '2022-09-30'
)
TARGET tx_fraud
FUNCTION fn_customer_cc_fd
IAM_ROLE default
AUTO OFF
MODEL_TYPE XGBOOST
OBJECTIVE 'binary:logistic'
PREPROCESSORS 'none'
HYPERPARAMETERS DEFAULT EXCEPT (NUM_ROUND '100')
SETTINGS (
S3_BUCKET '<thay thế bằng S3 bucket của bạn: zero-etl-approaches-file-store-*>',
S3_GARBAGE_COLLECT off,
MAX_RUNTIME 1200
);
```

![ML](/images/6.ML/5.png)

**Mô hình ML** được đặt tên là **cust_cc_txn_fd**, và hàm dự đoán là **fn_customer_cc_fd**. Câu lệnh **FROM** chỉ ra các cột đầu vào từ bảng lịch sử **public.cust_payment_tx_history**. Trường mục tiêu được đặt là **tx_fraud** — đây là biến mà chúng ta muốn dự đoán. **IAM_ROLE** được đặt là `default` vì cụm Redshift đã được cấu hình sẵn; nếu không, bạn cần cung cấp **IAM role ARN** cho cụm Redshift.

**MAX_RUNTIME** chỉ định thời gian huấn luyện tối đa (tính bằng giây). Giá trị mặc định là **5,400 giây (90 phút)**. Với lab này, ta dùng **1200 giây (20 phút)** để tiết kiệm thời gian.

Lệnh **CREATE MODEL** sẽ được thực thi **bất đồng bộ**, tức là chạy nền. Bạn có thể dùng lệnh sau để kiểm tra trạng thái mô hình:

```
SHOW MODEL cust_cc_txn_fd;
```


![ML](/images/6.ML/6.png)

![ML](/images/6.ML/7.png)

Từ kết quả trả về, ta thấy mô hình đã được phân loại đúng là **BinaryClassification** và có **train:error** rất thấp.

### Kiểm thử mô hình với tập dữ liệu kiểm tra

Chạy lệnh sau để lấy các dự đoán mẫu:

```
SELECT
tx_fraud ,
fn_customer_cc_fd(
TX_AMOUNT ,
TX_DURING_WEEKEND ,
TX_DURING_NIGHT ,
CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
TERMINAL_ID_NB_TX_1DAY_WINDOW ,
TERMINAL_ID_RISK_1DAY_WINDOW ,
TERMINAL_ID_NB_TX_7DAY_WINDOW ,
TERMINAL_ID_RISK_7DAY_WINDOW ,
TERMINAL_ID_NB_TX_30DAY_WINDOW ,
TERMINAL_ID_RISK_30DAY_WINDOW
)
FROM cust_payment_tx_history
WHERE cast(tx_datetime as date) >= '2022-10-01'
LIMIT 10;
```


![ML](/images/6.ML/8.png)

Ta có thể thấy một số giá trị dự đoán khớp với thực tế, một số thì không. Hãy so sánh kết quả dự đoán với nhãn thực tế:

```
SELECT
tx_fraud ,
fn_customer_cc_fd(
TX_AMOUNT ,
TX_DURING_WEEKEND ,
TX_DURING_NIGHT ,
CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
TERMINAL_ID_NB_TX_1DAY_WINDOW ,
TERMINAL_ID_RISK_1DAY_WINDOW ,
TERMINAL_ID_NB_TX_7DAY_WINDOW ,
TERMINAL_ID_RISK_7DAY_WINDOW ,
TERMINAL_ID_NB_TX_30DAY_WINDOW ,
TERMINAL_ID_RISK_30DAY_WINDOW
) AS prediction,
count(*) AS values
FROM public.cust_payment_tx_history
WHERE cast(tx_datetime as date) >= '2022-08-01'
GROUP BY 1,2;
```

![ML](/images/6.ML/9.png)

Chúng ta đã xác minh rằng mô hình hoạt động tốt. Hãy chuyển sang bước tiếp theo để tạo dự đoán từ dữ liệu streaming.