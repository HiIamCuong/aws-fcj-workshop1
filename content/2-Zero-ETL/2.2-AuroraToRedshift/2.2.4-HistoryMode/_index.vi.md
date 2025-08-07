---
title : "Chế độ lịch sử"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.2.4 </b> "
---

## Tổng quan
Với chế độ lịch sử (History Mode), bạn có thể cấu hình các Zero-ETL integrations để theo dõi mọi phiên bản (bao gồm cập nhật và xóa) của các bản ghi trong các bảng nguồn, trực tiếp trong Amazon Redshift. Sử dụng chế độ lịch sử, bạn có thể:

+ Thực hiện phân tích lịch sử,
+ Xây dựng các báo cáo xem lại,
+ Thực hiện phân tích xu hướng, và
+ Gửi các cập nhật gia tăng đến các ứng dụng downstream được xây dựng trên Amazon Redshift.

## Bật chế độ Lịch sử (History Mode)

1. Mở và kết nối với Redshift Query Editor
+ Mở **Amazon Redshift Query editor v2** (nếu chưa mở)
+ Kết nối với **Redshift Serverless data warehouse destination** được tạo trong workshop này bằng thông tin đăng nhập **awsuser/Awsuser123** như đã chỉ định.

![History Mode](/images/2.Zero-ETLIntegration/34.png)

![History Mode](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi câu SQL dưới đây để kiểm tra số lượng trong cơ sở dữ liệu đích **rdsmysql_zeroetl** được tạo từ integration. 
+ Mở một trình soạn thảo SQL khác trong query editor v2 để xem cơ sở dữ liệu mới tạo trong danh sách thả xuống. Ngoài ra, bạn có thể sử dụng [ký hiệu ba phần](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html), tức là **database_name.schema_name.object_name** để thực thi câu truy vấn dưới đây sử dụng **rdsmysql_zeroetl.seedingdemo.<table_name>** thay thế.

```
ALTER DATABASE rdsmysql_zeroetl INTEGRATION SET HISTORY_MODE = TRUE FOR TABLE seedingdemo.customer_info;
```


![History Mode](/images/2.Zero-ETLIntegration/108.png)

+ Điều này sẽ thay đổi trạng thái của bảng thành **Resync initiated**. Bạn có thể theo dõi trạng thái này trong bảng điều khiển Redshift dưới tab **Table Statistics** của Zero-ETL integration. Việc đồng bộ lại có thể mất thời gian tương tự như quá trình seeding (15-30 phút) cho bảng đó.

![History Mode](/images/2.Zero-ETLIntegration/109.png)

## Kết nối và thực hiện cập nhật và xóa trên cơ sở dữ liệu nguồn

{{% notice info %}}  
Trong bước này, bạn sẽ trải nghiệm **Chế độ Lịch sử** trong hành động. Tất cả các cập nhật/xóa thực hiện trên nguồn sẽ được đồng bộ gần như ngay lập tức, duy trì trạng thái của bảng bằng cách sử dụng 3 cột chế độ lịch sử sẽ được giải thích trong phần tiếp theo.
{{% /notice %}}

1. Để kết nối với Aurora MySQL instance của bạn
+ Điều hướng đến **EC2 console** 
+ Chọn **EC2 instance** đã cấp phát và nhấp vào **Connect**.

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. Trong trang **Connect**
+ Chọn **Session manager**
+ Chọn **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ Bạn sẽ được chuyển đến CLI của EC2 instance

3. Lấy Endpoint và Port của Aurora MySQL

+ Điều hướng đến **Aurora và RDS Console**
+ Chọn tính năng **Databases**
+ Chọn cơ sở dữ liệu có tiền tố tên **zero-etl-lab-sourcerdsmscluster-***
+ Chọn **Role Writer** dưới lựa chọn mặc định
+ Sao chép **endpoint** của writer

![Create Stack](/images/2.Zero-ETLIntegration/57.png)

4. Quay lại CLI **EC2 instance**
+ Sử dụng lệnh này để kết nối tới **RDS MySQL** từ **EC2 instance**

`mysql -h rds_mysql_instance_endpoint -P 6612 -u awsuser -p `

+ Mật khẩu để đăng nhập

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/58.png)

5. Khi đã kết nối với **Aurora MySQL instance**
+ Thực thi các câu lệnh DML dưới đây để thực hiện cập nhật/xóa

```
use seedingdemo;

update customer_info set job_title='History Mode Update Test' where customer_id=1;
commit;
update customer_info set job_title='History Mode Update Test-Second Immediate Update' where customer_id=1;
commit;
delete from customer_info where customer_id=1;
commit;
```


![History Mode](/images/2.Zero-ETLIntegration/110.png)

+ Xác minh trạng thái **Resync initiated** (áp dụng cho giao dịch đầu tiên sau khi bật chế độ lịch sử) đã thay đổi thành **Synced** trong bảng điều khiển Redshift dưới tab **Table Statistics** của Zero-ETL integration.

![History Mode](/images/2.Zero-ETLIntegration/109.png)

## Xác minh trong đích Redshift

1. Quay lại **Amazon Redshift Query editor v2**
+ Chạy các câu truy vấn kiểm tra và giám sát sau
+ Chọn cơ sở dữ liệu đích **rdsmysql_zeroetl** được tạo từ integration.

```
select * from seedingdemo.customer_info order by customer_id asc, _record_create_time desc, _record_delete_time desc;
```


![History Mode](/images/2.Zero-ETLIntegration/111.png)

Khi chế độ lịch sử được bật, các cột sau sẽ được thêm vào bảng đích của bạn để theo dõi các thay đổi từ nguồn. Mọi bản ghi nguồn bị xóa hoặc thay đổi sẽ tạo ra một bản ghi mới trong đích, dẫn đến nhiều dòng tổng cộng hơn trong bảng đích với các phiên bản bản ghi khác nhau. Các bản ghi không bị xóa khỏi bảng đích khi bị xóa hoặc sửa đổi trong nguồn. Bạn có thể quản lý bảng đích bằng cách xóa các bản ghi không hoạt động.

| Tên Cột               | Kiểu Dữ Liệu | Mô Tả                                                                 |
|-----------------------|--------------|------------------------------------------------------------------------|
| _record_is_active     | Boolean      | Chỉ ra liệu bản ghi trong bảng đích hiện tại có đang hoạt động trong nguồn không. `True` chỉ ra bản ghi đang hoạt động. |
| _record_create_time   | Timestamp    | Thời gian bắt đầu (UTC) khi bản ghi nguồn có hiệu lực.                      |
| _record_delete_time   | Timestamp    | Thời gian kết thúc (UTC) khi bản ghi nguồn bị cập nhật hoặc xóa.            |

{{% notice info %}}  
**Chúc mừng!** Bạn đã hoàn thành thành công một ví dụ về **CHẾ ĐỘ LỊCH SỬ** trong **Zero-ETL integrations**
{{% /notice %}}
