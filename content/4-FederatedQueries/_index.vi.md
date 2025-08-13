---
title : "Truy vấn Liên kết (Federated Queries) trong Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---

Bằng cách sử dụng **truy vấn liên kết (federated queries)** trong Amazon Redshift, bạn có thể truy vấn và phân tích dữ liệu trên các cơ sở dữ liệu hoạt động, kho dữ liệu và hồ dữ liệu. Với tính năng **Federated Query**, bạn có thể tích hợp các truy vấn từ Amazon Redshift trên dữ liệu trực tiếp trong các cơ sở dữ liệu ngoài với các truy vấn trên môi trường **Amazon Redshift** và **Amazon S3** của mình.

Truy vấn liên kết có thể hoạt động với các cơ sở dữ liệu ngoài như:

- **Amazon RDS for PostgreSQL**
- **Amazon Aurora phiên bản tương thích PostgreSQL**
- **Amazon RDS for MySQL**
- **Amazon Aurora phiên bản tương thích MySQL**

Bạn có thể sử dụng truy vấn liên kết để kết hợp dữ liệu trực tiếp như một phần của các ứng dụng báo cáo và phân tích kinh doanh (BI). Ví dụ, để giúp việc nạp dữ liệu vào Amazon Redshift trở nên đơn giản hơn, bạn có thể sử dụng truy vấn liên kết để:

- Truy vấn trực tiếp các cơ sở dữ liệu hoạt động
- Áp dụng các phép biến đổi nhanh chóng
- Nạp dữ liệu vào các bảng đích mà không cần quy trình ETL (Extract, Transform, Load) phức tạp

Để giảm thiểu việc di chuyển dữ liệu qua mạng và cải thiện hiệu suất, Amazon Redshift phân bổ một phần việc xử lý truy vấn liên kết trực tiếp đến các cơ sở dữ liệu hoạt động từ xa. Ngoài ra, Redshift còn tận dụng khả năng xử lý song song của mình để hỗ trợ thực thi các truy vấn này khi cần thiết.

## Cách hoạt động của Truy vấn Liên kết

Khi thực thi một truy vấn liên kết, Amazon Redshift thực hiện các bước sau:

1. **Nút leader** tạo kết nối khách hàng đến phiên bản RDS hoặc Aurora DB để lấy **metadata của bảng**.
2. Một **nút tính toán** sẽ phát hành các truy vấn con với các điều kiện lọc được đẩy xuống và lấy các dòng kết quả.
3. Amazon Redshift sau đó **phân phối các dòng kết quả** đến các nút tính toán để xử lý thêm.

Quy trình này cho phép truy vấn dữ liệu hiệu quả và có khả năng mở rộng từ nhiều nguồn dữ liệu khác nhau.

![Architecture](/images/4.FederatedQuery/14.png)