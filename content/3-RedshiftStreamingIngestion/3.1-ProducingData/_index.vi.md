---
title : "Tạo dữ liệu"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---

Trong bài lab này, chúng ta sẽ gửi dữ liệu đến Kinesis Data Stream bằng cách sử dụng kết hợp giữa EventBridge và Lambda.

Chúng tôi đã cung cấp sẵn ba thành phần cho bạn như một phần của bài lab này:

+ Một Kinesis Data Stream - **cust-payment-txn-stream**

+ Một EventBridge rule - **zero-etl-lab-EventRule**

+ Một Lambda function - **GenStreamData-zero-etl-lab**

Hàm Lambda sẽ tạo dữ liệu giao dịch tài chính giả lập của khách hàng để gửi đến Kinesis Data Stream và được kích hoạt bởi EventBridge để tạo dữ liệu ngẫu nhiên mỗi phút. Hãy thiết lập luồng này và kiểm tra xem stream đã nhận được dữ liệu chưa.

1. Truy cập [EventBridge](https://console.aws.amazon.com/events/home) trong AWS Console

2. Chọn **Rules**

3. Chọn và nhấp vào rule bắt đầu với **zero-etl-lab-EventRule**

4. Nhấn nút **Enable**

![ProducingData](/images/3.RedshiftStreamingIngestion/1.png)

![ProducingData](/images/3.RedshiftStreamingIngestion/2.png)

![ProducingData](/images/3.RedshiftStreamingIngestion/3.png)

EventBridge giờ sẽ kích hoạt hàm Lambda để tạo dữ liệu mỗi phút.

5. Bây giờ truy cập vào [Kinesis](https://console.aws.amazon.com/kinesis/home)

6. Chọn **Data streams** ở menu bên trái

7. Chọn và nhấn vào stream có tên **cust-payment-txn-stream**

8. Chuyển đến tab **Monitoring** của stream

{{% notice info %}}
Lưu ý: Nếu bạn không thấy số liệu được cập nhật hoặc bảng điều khiển trống, có thể mất đến 15 phút để cập nhật. Hãy tiếp tục phần lab khác và quay lại phần này sau.
{{% /notice %}}

![ProducingData](/images/3.RedshiftStreamingIngestion/6.png)

9. Khi bạn thấy dữ liệu bắt đầu chảy vào stream, hãy quay lại [EventBridge](https://console.aws.amazon.com/events/home) và vô hiệu hóa rule đã bật trước đó.

![ProducingData](/images/3.RedshiftStreamingIngestion/7.png)

![ProducingData](/images/3.RedshiftStreamingIngestion/8.png)

Chúc mừng, bạn đã thiết lập thành công một Kinesis Data Stream nhận dữ liệu từ Lambda được kích hoạt bởi EventBridge. Hãy tiếp tục tạo một view của dữ liệu này trong Redshift.
