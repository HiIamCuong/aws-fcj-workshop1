---
title : "Giới thiệu tích hợp không ETL (zero-ETL)"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2. </b> "
---

## Định nghĩa
**Tích hợp không ETL (zero-ETL):** là một tập hợp các tích hợp giúp loại bỏ hoặc giảm thiểu nhu cầu xây dựng quy trình dữ liệu ETL. extract (trích xuất), transform (chuyển đổi) và load (tải) (ETL) là quy trình kết hợp, làm sạch và chuẩn hóa dữ liệu từ các nguồn khác nhau để sẵn sàng cho khối lượng công việc phân tích, AI và ML. Các quy trình ETL truyền thống tốn nhiều thời gian và phức tạp để phát triển, duy trì và điều chỉnh quy mô. Thay vào đó, tích hợp không ETL tạo điều kiện thuận lợi cho việc di chuyển dữ liệu point-to-point (điểm nối điểm) mà không cần tạo quy trình dữ liệu ETL. Tích hợp không ETL cũng có thể cho phép truy vấn qua các lô cốt dữ liệu mà không cần di chuyển dữ liệu.

## Các lợi ích mà Tích hợp không ETL mang lại:
- Giảm độ phức tạp của hệ thống khi phải ETL.
- Giảm phụ phí khi thực hiện quy trình ETL.
- Không làm trì hoãn thời gian cho phân tích, AI, ML.
- Tăng tính linh hoạt vì nó làm đơn giản hóa kiến trúc dữ liệu, cho phép giảm nỗ lực cũng như sử dụng dữ liệu mới và không phải xử lý lượng lớn dữ liệu.
- Thu thập thông tin chuyên sâu nhanh hơn do các quy trình ETL truyền thống thường bao gồm đến bản cập nhật hàng loạt định kỳ, từ đó trì hoãn tính sẵn có của dữ liệu. Tích hợp không ETL lại cung cấp quyền truy cập dữ liệu theo thời gian thực hoặc gần thời gian thực, đảm bảo dữ liệu mới hơn để phân tích, cho công nghệ AI/ML và báo cáo, tăng trải nghiệm người dùng.

## Ứng dụng của zero-ETL:

- Doanh nghiệp cần nhanh chóng tải nhập và phân tích các loại dữ liệu khác nhau để đưa ra quyết định trong thời gian thực.
- Nền tảng truyền dữ liệu và hàng đợi tin nhắn truyền dữ liệu thời gian thực từ một số nguồn.

Theo truyền thống, quá trình di chuyển dữ liệu từ cơ sở dữ liệu hoạt động và giao dịch vào data warehouse và data lake luôn đòi hỏi một giải pháp ETL phức tạp. Ngày nay, khả năng tích hợp không ETL có thể hoạt động như một công cụ sao chép dữ liệu, ngay lập tức sao chép dữ liệu từ cơ sở dữ liệu hoạt động, cơ sở dữ liệu giao dịch và các ứng dụng vào data warehouse và data lake.

![Architecture](/images/2.Zero-ETLIntegration/147.png)