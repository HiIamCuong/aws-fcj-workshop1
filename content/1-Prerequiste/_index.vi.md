---
title : "Các bước chuẩn bị"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

{{% notice info %}}
Trước hết bạn cần đăng nhập vào tài khoản **AWS Console Management** có **quyền Administrator** và chọn **region** **N.Virginia (us-east-1)** hoặc **Oregon (us-west-2)** 
{{% /notice %}}

1. Tạo Stack

Ấn vào [đây](https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=zero-etl-lab&templateURL=https://redshift-demos.s3.amazonaws.com/zetl/approaches/zeroetl.yaml) này để vào trang cấu hình Stack trong **AWS CloudFormation**.

(**Stack** trong **AWS CloudFormation** có chức năng lưu trữ cấu hình của các dịch vụ AWS và có thể tạo, chạy và xóa đồng loạt các dịch vụ AWS khi tạo và xóa **Stack**)

2. Sau khi nhấn vào Link ta vào trang bước 1
+ Nhấn **Next**

![Create Stack](/images/1.prerequisite/1.png)

3. Tại bước 2
+ Nhấn **Next**

![Create Stack](/images/1.prerequisite/2.png)

![Create Stack](/images/1.prerequisite/3.png)

4. Tại bước 3
+ Tích vào **I acknowledge that AWS CloudFormation might create IAM resources with custom name** 
+ Tích vào **I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND**
+ Nhấn **Next**

![Create Stack](/images/1.prerequisite/4.png)

![Create Stack](/images/1.prerequisite/5.png)

![Create Stack](/images/1.prerequisite/6.png)

5. Tại bước 4
+ Kiểm tra lại thông tin
+ Nhấn **Next**

![Create Stack](/images/1.prerequisite/7.png)

![Create Stack](/images/1.prerequisite/8.png)

![Create Stack](/images/1.prerequisite/9.png)

![Create Stack](/images/1.prerequisite/10.png)

![Create Stack](/images/1.prerequisite/11.png)

6. Đợi để Stack tạo xong các tài nguyên

![Create Stack](/images/1.prerequisite/12.png)

7. Hoàn thành

![Create Stack](/images/1.prerequisite/13.png)

{{% notice info %}}
Đến bước này đã xong bước chuẩn bị. Bạn có thể đi đến [Part 2](2-Zero-ETL/). Phần dưới đây là để giải thích thêm về các tài nguyên Stack này tạo ra
{{% /notice %}}

- Một **VPC** với nhiều **subnet**, **routes table** và các tài nguyên mạng khác như **Security Group** để sử dụng **EC2**, **RDS**, **Aurora**

![Create Stack](/images/1.prerequisite/14.png)

- Một **bucket** của **Amazon S3** với tên *zero-etl-approaches-file-store-** để chứa thư mục Machine Learning

![Create Stack](/images/1.prerequisite/15.png)

- **IAM Roles** với các quyền cần thiết để thực thi truy vấn trên **Amazon RDS** và **Amazon Redshift**.

![Create Stack](/images/1.prerequisite/35.png)

- Một **EC2 instance** để kết nối và thực thi truy vấn trên **Aurora MySQL** và **Aurora PostgreSQL**

![Create Stack](/images/1.prerequisite/16.png)

![Create Stack](/images/1.prerequisite/17.png)

![Create Stack](/images/1.prerequisite/18.png)

![Create Stack](/images/1.prerequisite/19.png)

![Create Stack](/images/1.prerequisite/20.png)

- **Glue crawler** tên *CustPaymentTxHistory* để lấy dữ liệu từ **Amazon S3**

![Create Stack](/images/1.prerequisite/21.png)

- và **Glue database** tên *spectrumdb*

![Create Stack](/images/1.prerequisite/22.png)

- **Kinesis Data Stream** tên *cust-payment-txn-stream*.

![Create Stack](/images/1.prerequisite/23.png)

- **Lambda function** tên *GenStreamData-zero-etl-lab* thứ sẽ gửi data đến **Kinesis Data Stream**

![Create Stack](/images/1.prerequisite/24.png)

- **AWS Eventbridge Rule** tên *zero-etl-lab-EventRule-* * để thực thi lambda function (ở trên) mỗi 1 phút khi được kích hoạt.

![Create Stack](/images/1.prerequisite/25.png)

-**(Nested Stack)** **Amazon Aurora MySQL 3.05.0 (hoặc phiên bản cao hơn) (tương thích với MySQL 8.0.32 hoặc cao hơn)** của **DbInstanceClass: Serverless V2**, kèm theo một **DbClusterParameterGroup** mới, mỗi thành phần đều có tiền tố tên là: *zero-etl-lab-sourceamscluster-* *.

![Create Stack](/images/1.prerequisite/27.png)

![Create Stack](/images/1.prerequisite/26.png)

- **(NESTED Stack) Amazon RDS MySQL 8.0.36 (or higher version)** **DbInstance** tên *prefix zero-etl-lab-sourcerdsmscluster-* *.

![Create Stack](/images/1.prerequisite/28.png)

![Create Stack](/images/1.prerequisite/30.png)

- **(NESTED Stack) Amazon Aurora PostgreSQL 13.9 DbInstanceClass: Serverless V1**, với **cluster/instance** tên *aurorapg*.

![Create Stack](/images/1.prerequisite/34.png)

![Create Stack](/images/1.prerequisite/33.png)