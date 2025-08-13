---
title : "Redshift Streaming Ingestion"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 3. </b> "
---

**Amazon Redshift Streaming Ingestion** là một công cụ mạnh mẽ giúp bạn tiếp nhận và phân tích dữ liệu phát trực tuyến với khả năng xử lý thông lượng cao và độ trễ thấp, phù hợp cho những yêu cầu về phân tích dữ liệu thời gian thực. Dưới đây là các điểm nổi bật của tính năng này:

## 1. Thông Lượng Cao, Độ Trễ Thấp
- Amazon Redshift Streaming Ingestion có thể xử lý hàng trăm megabyte dữ liệu mỗi giây từ các nguồn phát trực tuyến khác nhau.
- Điều này giúp bạn thu thập thông tin từ dữ liệu ngay lập tức, hỗ trợ các quyết định kinh doanh dựa trên dữ liệu gần như thời gian thực.

## 2. Quy Trình Nhập Dữ Liệu Đơn Giản
- Bạn có thể nhập trực tiếp dữ liệu phát trực tuyến từ **Kinesis Data Streams** hoặc **Amazon Managed Streaming for Apache Kafka (MSK)** vào kho dữ liệu Amazon Redshift mà không cần phải lưu trữ tạm thời trên **Amazon S3**.
- Điều này giúp loại bỏ bước ETL (Extract, Transform, Load) phức tạp, tiết kiệm thời gian và tài nguyên.

## 3. Xử Lý Dữ Liệu Phát Trực Tuyến Mà Không Cần ETL
- Không cần phải sử dụng ETL truyền thống, giúp tiết kiệm công sức trong việc chuyển đổi dữ liệu trước khi nhập vào hệ thống.
- Có thể làm việc trực tiếp với dữ liệu bán cấu trúc qua kiểu dữ liệu **SUPER** trong Amazon Redshift, mang lại sự linh hoạt cao trong việc xử lý dữ liệu không cấu trúc.

## 4. Phân Tích Dữ Liệu Phong Phú Với SQL
- Bạn có thể sử dụng **SQL** quen thuộc để thực hiện các truy vấn dữ liệu trên dữ liệu phát trực tuyến.
- Hỗ trợ các **materialized views** (MV) giúp bạn nhanh chóng xây dựng các chế độ xem dữ liệu từ dòng dữ liệu trực tiếp mà không cần phải làm lại tính toán mỗi lần.

## 5. Trực Quan Hóa Dữ Liệu trong Thời Gian Thực
- Dữ liệu được nhập vào có thể được trực quan hóa nhanh chóng qua các công cụ BI như **Amazon QuickSight**.
- Bạn có thể tạo ra các hình ảnh, biểu đồ hoặc báo cáo để làm nổi bật các xu hướng, mẫu dữ liệu trong thời gian thực.
- **Amazon QuickSight Q**, hỗ trợ bởi machine learning (ML), còn giúp bạn đặt câu hỏi và nhận câu trả lời từ dữ liệu thông qua hình ảnh trực quan.

## 6. Dễ Dàng Quản Lý Dòng Dữ Liệu
- Với khả năng quản lý tự động của Amazon Redshift, bạn có thể tạo và quản lý các **pipeline ELT** trực tiếp trong hệ thống.
- Việc sử dụng **hàm do người dùng định nghĩa (UDF)** và **thủ tục lưu trữ** giúp tối ưu hóa quy trình xử lý dòng dữ liệu.

---

**Tóm lại**, Amazon Redshift Streaming Ingestion mang lại một giải pháp mạnh mẽ, nhanh chóng và dễ dàng cho việc xử lý và phân tích dữ liệu phát trực tuyến, giúp bạn tạo ra các quyết định dữ liệu thông minh trong thời gian thực mà không cần phải lo lắng về các công đoạn xử lý phức tạp.

![architecture](/images/3.RedshiftStreamingIngestion/15.png)