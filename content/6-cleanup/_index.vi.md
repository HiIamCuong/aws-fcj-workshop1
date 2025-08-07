+++
title = "Dọn dẹp tài nguyên  "
date = 2021
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

{{% notice info %}}   
Bạn **PHẢI** dọn dẹp trong **tài khoản AWS của chính bạn** để tránh các chi phí phát sinh không kiểm soát.
{{% /notice %}}

Chúng ta sẽ thực hiện các bước sau để xóa tài nguyên đã tạo trong bài thực hành này.

# 1. Xóa tích hợp Zero-ETL
Xóa các tích hợp zero-etl được tạo như một phần của hội thảo này có tên **zero-etl-***, thực hiện theo các bước dưới đây để xóa:

+ Trên **bảng điều khiển Amazon RDS**, chọn **Zero-ETL integrations** trong bảng điều hướng.
+ Chọn **zero-ETL integration** bạn muốn xóa và nhấn **Delete**. Thực hiện điều này cho **TẤT CẢ** các tích hợp đã tạo.
+ Để xác nhận việc xóa, nhập vào ô `confirm` sau đó chọn **Delete**.

![Clean](/images/7.clean/1.png)

![Clean](/images/7.clean/2.png)

![Clean](/images/7.clean/3.png)

# 2. Xóa Stack 

{{% notice info %}}  
Bạn cần kiểm tra và **làm trống** **S3 bucket** có tên bắt đầu bằng **zero-etl-*** trước vì CloudFormation không thể **làm trống** bucket. Điều này sẽ gây lỗi khi xóa stack.
{{% /notice %}}

![Clean](/images/7.clean/8.png)

1. Làm trống S3 bucket

+ Truy cập **bảng điều khiển S3**, chọn **Bucket**
+ Chọn bucket có tên bắt đầu bằng **zero-etl-*** 
+ Chọn **Empty**

![Clean](/images/7.clean/6.png)

+ Nhập `permanently delete`
+ Chọn **Empty**

![Clean](/images/7.clean/5.png)

![Clean](/images/7.clean/7.png)

2. Xóa Stack

+ Truy cập **bảng điều khiển CloudFormation**
+ Chọn **Stacks**
+ Chọn **zero-etl-lab** (không có chữ NESTED trong tên)

![Clean](/images/7.clean/4.png)

![Clean](/images/7.clean/11.png)
