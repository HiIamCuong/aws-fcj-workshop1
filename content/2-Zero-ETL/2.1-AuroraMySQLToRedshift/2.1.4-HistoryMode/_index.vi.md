---
title : "Chế độ lịch sử"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.1.4 </b> "
---

## Tổng quan

Với **chế độ lịch sử (History Mode)**, bạn có thể cấu hình các tích hợp Zero-ETL để theo dõi mọi phiên bản (bao gồm cập nhật và xóa) của các bản ghi trong bảng nguồn, trực tiếp trên **Amazon Redshift**. Sử dụng chế độ lịch sử, bạn có thể:

+ Thực hiện phân tích lịch sử,  
+ Xây dựng báo cáo theo thời gian,  
+ Phân tích xu hướng, và  
+ Gửi các bản cập nhật gia tăng đến các ứng dụng phụ trợ sử dụng Amazon Redshift  

## Bật Chế Độ Lịch Sử

1. Mở và kết nối Redshift Query Editor  
+ Mở **Amazon Redshift Query editor v2** (nếu chưa mở)  
+ Kết nối với **Redshift Serverless datawarehouse destination** được tạo trong workshop, sử dụng **awsuser/Awsuser123** như hình dưới:

![History Mode](/images/2.Zero-ETLIntegration/34.png)  
![History Mode](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi SQL sau để kiểm tra số bản ghi trong cơ sở dữ liệu đích **auroramysql_zeroetl** được tạo từ tích hợp.  
+ Mở một trình SQL khác trong Query editor v2 để thấy cơ sở dữ liệu mới. Hoặc bạn có thể dùng [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html):  
**database_name.schema_name.object_name**, ví dụ: **auroramysql_zeroetl.seedingdemo.region**

```
ALTER DATABASE auroramysql_zeroetl INTEGRATION SET HISTORY_MODE = TRUE FOR TABLE seedingdemo.region;
```

![History Mode](/images/2.Zero-ETLIntegration/52.png)

+ Điều này sẽ đưa bảng vào trạng thái **Resync initiated**. Có thể theo dõi tại Redshift console dưới tab **Table Statistics**. Quá trình đồng bộ lại có thể mất 15–30 phút.

![History Mode](/images/2.Zero-ETLIntegration/53.png)

## Kết nối và chạy cập nhật, xóa trên CSDL nguồn

{{% notice info %}}  
Trong bước này, bạn sẽ trải nghiệm **Chế độ lịch sử** hoạt động. Mọi cập nhật/xóa thực hiện trên nguồn sẽ được đồng bộ gần như tức thì, đồng thời lưu giữ trạng thái bảng bằng 3 cột chế độ lịch sử được giải thích ở phần sau.
{{% /notice %}}

1. Kết nối đến Aurora MySQL instance  
+ Truy cập **EC2 Console**  
+ Chọn **EC2 instance** đã được khởi tạo và click **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. Trên trang **Connect**  
+ Chọn **Session manager**  
+ Click **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ Bạn sẽ truy cập CLI của EC2 instance

3. Lấy Endpoint và Port của Aurora MySQL  
+ Vào **Aurora/RDS Console**  
+ Vào tab **Databases**  
+ Chọn database có prefix tên **zero-etl-lab-sourceamscluster-***  
+ Chọn **Role: Writer** trong mặc định  
+ Sao chép **endpoint** của writer

![Create Stack](/images/2.Zero-ETLIntegration/3.png)

4. Trở lại CLI trong EC2 instance  
+ Dùng lệnh sau để kết nối đến Aurora MySQL từ EC2:

`mysql -h <aurora_mysql_writer_endpoint> -P 3306 -u awsuser -p`

+ Mật khẩu để đăng nhập:

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/5.png)

5. Khi đã kết nối tới Aurora MySQL instance  
+ Thực thi các câu lệnh DML sau để cập nhật và xóa:

```
use seedingdemo;

update region set r_comment='History Mode Update Test' where r_regionkey=1;
commit;
update region set r_comment='History Mode Update Test-Second Immediate Update' where r_regionkey=1;
commit;
delete from region where r_regionkey=1;
commit;
```

![History Mode](/images/2.Zero-ETLIntegration/54.png)

+ Kiểm tra rằng trạng thái **Resync initiated** (áp dụng với giao dịch đầu tiên sau khi bật chế độ lịch sử) đã chuyển sang **Synced** trong Redshift console tab **Table Statistics**

![History Mode](/images/2.Zero-ETLIntegration/55.png)

## Xác minh trong Redshift target

1. Quay lại Redshift Query editor v2  
+ Chạy truy vấn sau để xác minh và theo dõi  
+ Chọn cơ sở dữ liệu đích **auroramysql_zeroetl** được tích hợp

```
select * from seedingdemo.region order by _record_create_time desc, _record_delete_time desc;
```

![History Mode](/images/2.Zero-ETLIntegration/56.png)

Khi bật chế độ lịch sử, các cột sau sẽ được thêm vào bảng đích để theo dõi thay đổi từ nguồn.  
Mọi bản ghi bị xóa hoặc thay đổi sẽ tạo một bản ghi mới trên Redshift, do đó số dòng sẽ nhiều hơn so với nguồn.  
Bản ghi không bị xóa khỏi Redshift khi bị xóa trên nguồn. Bạn có thể tự quản lý bằng cách xóa các bản ghi không còn hiệu lực.

| Column Name           | Data Type | Description                                                                 |
|-----------------------|-----------|-----------------------------------------------------------------------------|
| _record_is_active     | Boolean   | Cho biết bản ghi trong đích có đang còn hiệu lực không. `True` nghĩa là còn hiệu lực. |
| _record_create_time   | Timestamp | Thời điểm (UTC) bản ghi bắt đầu hiệu lực trong nguồn.                      |
| _record_delete_time   | Timestamp | Thời điểm (UTC) bản ghi bị cập nhật hoặc xóa từ nguồn.                     |

{{% notice info %}}  
**Chúc mừng!** Bạn đã hoàn tất phần trình diễn **HISTORY MODE** trong **Zero-ETL integrations**
{{% /notice %}}
