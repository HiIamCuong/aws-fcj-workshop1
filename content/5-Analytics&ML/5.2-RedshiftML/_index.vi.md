---
title : "Redshift Machine Learning"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 5.2 </b> "
---

# Amazon Redshift ML

Amazon Redshift ML giúp các nhà phân tích dữ liệu và nhà phát triển cơ sở dữ liệu dễ dàng tạo, huấn luyện và áp dụng các mô hình học máy bằng cách sử dụng các lệnh SQL quen thuộc trong kho dữ liệu Amazon Redshift. Với Redshift ML, bạn có thể tận dụng Amazon SageMaker – một dịch vụ học máy được quản lý toàn diện – mà không cần học thêm công cụ hoặc ngôn ngữ mới. Bạn chỉ cần sử dụng các câu lệnh SQL để tạo và huấn luyện các mô hình học máy từ Amazon SageMaker bằng dữ liệu trong Redshift, rồi sử dụng các mô hình này để đưa ra dự đoán.

## Ví dụ về Trường hợp Sử dụng
Ví dụ, bạn có thể sử dụng dữ liệu về khả năng giữ chân khách hàng trong Redshift để huấn luyện một mô hình phát hiện khách hàng có nguy cơ rời bỏ, sau đó áp dụng mô hình này trong các bảng điều khiển (dashboard) cho đội ngũ tiếp thị để tạo ra các ưu đãi giữ chân phù hợp. Redshift ML cung cấp mô hình dưới dạng các hàm SQL trong kho dữ liệu Redshift, giúp bạn dễ dàng sử dụng trực tiếp trong các truy vấn và báo cáo.

### Tính Năng Nổi Bật

#### Không Cần Kinh Nghiệm Học Máy Trước Đó
Vì Redshift ML cho phép bạn sử dụng SQL tiêu chuẩn, bạn có thể dễ dàng phát triển các trường hợp sử dụng mới cho dữ liệu phân tích của mình. Redshift ML cung cấp khả năng tích hợp đơn giản, được tối ưu hóa và bảo mật giữa Redshift và Amazon SageMaker, đồng thời hỗ trợ suy luận (inference) ngay bên trong cụm Redshift. Điều này giúp bạn dễ dàng sử dụng các dự đoán từ mô hình học máy trong các truy vấn và ứng dụng mà không cần quản lý một điểm cuối (endpoint) suy luận riêng biệt. Dữ liệu huấn luyện cũng được bảo mật toàn diện với mã hóa.

#### Sử Dụng Học Máy trên Dữ Liệu Redshift bằng SQL Tiêu Chuẩn
Để bắt đầu, bạn sử dụng lệnh SQL `CREATE MODEL` trong Redshift và chỉ định dữ liệu huấn luyện dưới dạng bảng hoặc câu lệnh `SELECT`. Redshift ML sẽ biên dịch và nhập mô hình đã huấn luyện vào trong kho dữ liệu Redshift, đồng thời tạo một hàm SQL để suy luận (inference) có thể được sử dụng ngay lập tức trong các truy vấn. Redshift ML sẽ tự động xử lý tất cả các bước cần thiết để huấn luyện và triển khai mô hình.

#### Phân Tích Dự Đoán với Amazon Redshift
Với Redshift ML, bạn có thể tích hợp các dự đoán như phát hiện gian lận, đánh giá rủi ro và dự đoán rời bỏ khách hàng trực tiếp vào các truy vấn và báo cáo. Bạn có thể chạy hàm SQL “churn prediction” (dự đoán khách hàng rời bỏ) trên dữ liệu khách hàng mới trong kho dữ liệu theo định kỳ để dự đoán những khách hàng có nguy cơ rời bỏ và cung cấp thông tin này cho các nhóm bán hàng và tiếp thị nhằm thực hiện các hành động phòng ngừa – như gửi ưu đãi giữ chân phù hợp.

#### Mang Theo Mô Hình Của Riêng Bạn (BYOM)
Redshift ML hỗ trợ sử dụng mô hình BYOM (Bring Your Own Model) để suy luận cục bộ hoặc từ xa. Bạn có thể sử dụng mô hình được huấn luyện bên ngoài Redshift bằng Amazon SageMaker để suy luận trong cơ sở dữ liệu Redshift. Bạn có thể nhập các mô hình huấn luyện bởi SageMaker Autopilot hoặc các mô hình SageMaker tùy chỉnh để thực hiện suy luận tại chỗ (local inference). Ngoài ra, bạn cũng có thể gọi các mô hình học máy tùy chỉnh từ xa được triển khai tại các endpoint của SageMaker. Bạn có thể sử dụng bất kỳ mô hình học máy nào của SageMaker chấp nhận và trả về dữ liệu dạng văn bản (text) hoặc CSV để thực hiện suy luận từ xa.
