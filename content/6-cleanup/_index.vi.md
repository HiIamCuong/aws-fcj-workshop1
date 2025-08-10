+++
title = "Dọn dẹp tài nguyên  "
date = 2021
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

{{% notice info %}}  
Bạn **PHẢI** dọn dẹp trong **tài khoản AWS của bạn** để tránh các chi phí không mong muốn.  
{{% /notice %}}

Chúng tôi sẽ thực hiện các bước sau để xóa các tài nguyên chúng tôi đã tạo trong bài tập này.

## 1. Xóa tích hợp Zero-ETL
Xóa các tích hợp zero-etl được tạo ra trong khuôn khổ của workshop này có tên **zero-etl-***, làm theo các bước dưới đây để xóa:

+ Trong **Amazon RDS console**, chọn **Zero-ETL integrations** trong bảng điều hướng.
+ Chọn **Zero-ETL integration** mà bạn muốn xóa và chọn **Delete**. Làm điều này cho **TẤT CẢ** các tích hợp đã tạo.
+ Để xác nhận xóa, gõ vào ô `confirm` rồi chọn **Delete**.

![Clean](/images/7.clean/1.png)

![Clean](/images/7.clean/2.png)

![Clean](/images/7.clean/3.png)

## 2. Xóa Stack

{{% notice info %}}  
Bạn cần kiểm tra và **dọn sạch** **S3 bucket** có tên **zero-etl-*** trước, vì CloudFormation không thể **dọn sạch** bucket. Điều này sẽ gây lỗi khi xóa **stack**. Nếu bạn không **empty** **S3 bucket** trước mà vẫn xóa **stack** điều này sẽ xảy ra
{{% /notice %}}

![Clean](/images/7.clean/8.png)

1. Dọn sạch S3 bucket

+ Đi đến **S3 console** và chọn **Bucket**
+ Chọn bucket có tiền tố tên **zero-etl-*** 
+ Chọn **Empty**

![Clean](/images/7.clean/6.png)

+ Gõ `permanently delete`
+ Chọn **Empty**

![Clean](/images/7.clean/5.png)

![Clean](/images/7.clean/7.png)

2. Xóa Stack

+ Đi đến **CloudFormation Console**
+ Chọn **Stacks**
+ Chọn **zero-etl-lab** (không có chữ NESTED trong tên)

![Clean](/images/7.clean/4.png)

![Clean](/images/7.clean/11.png)

## 3. Xóa DB snapshots

+ Đi đến **Aurora và RDS console**
+ Chọn snapshots trong bảng điều hướng bên trái
+ Kiểm tra và **xóa** tất cả các snapshots **manual snapshots** và **system snapshots**

![Clean](/images/7.clean/12.png)

{{% notice info %}}  
**Chúc mừng** bạn đã hoàn thành việc **dọn dẹp** và hoàn thành buổi hội thảo. Cảm ơn bạn đã dành thời gian cho phần này.
{{% /notice %}}