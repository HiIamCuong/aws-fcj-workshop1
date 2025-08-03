---
title : "Chế độ lịch sử"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.3.4 </b> "
---

## Tổng Quan

Với chế độ lịch sử (History Mode), bạn có thể cấu hình các Zero-ETL integrations của mình để theo dõi mọi phiên bản (bao gồm cả các bản cập nhật và xóa) của các bản ghi trong các bảng nguồn, trực tiếp trên Amazon Redshift. Khi sử dụng chế độ lịch sử, bạn có thể:

+ Thực hiện phân tích lịch sử,
+ Tạo báo cáo hồi cứu,
+ Thực hiện phân tích xu hướng, và
+ Gửi các bản cập nhật gia tăng đến các ứng dụng phía dưới được xây dựng trên Amazon Redshift.

## Bật Chế Độ Lịch Sử

1. Mở và Kết Nối Redshift Query Editor
+ Mở **Amazon Redshift Query Editor v2** (nếu chưa mở)
+ Kết nối với **Redshift Serverless datawarehouse destination** đã được tạo trong workshop này bằng thông tin tài khoản **awsuser/Awsuser123** như đã chỉ ra.

![History Mode](/images/2.Zero-ETLIntegration/34.png)

![History Mode](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi câu lệnh SQL dưới đây để kiểm tra số lượng trong cơ sở dữ liệu đích **aurorapostgres_zeroetl** được tạo từ integration.
+ Mở một trình soạn thảo SQL khác trong query editor v2 để xem cơ sở dữ liệu mới tạo trong danh sách thả xuống. Ngoài ra, bạn có thể sử dụng [cú pháp ba phần](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html), tức là **database_name.schema_name.object_name** để thực thi câu truy vấn dưới đây bằng cách sử dụng **aurorapostgres_zeroetl.public.<table_name>**.

```
ALTER DATABASE aurorapostgres_zeroetl INTEGRATION SET HISTORY_MODE = TRUE FOR TABLE public.customer_info;
```


![History Mode](/images/2.Zero-ETLIntegration/108.png)

+ Điều này sẽ thay đổi trạng thái bảng thành **Resync initiated**. Bạn có thể theo dõi điều này trong bảng điều khiển Redshift dưới tab **Table Statistics** của Zero-ETL integration. Quá trình resync có thể mất thời gian tương tự như quá trình seeding (15-30 phút) đối với bảng đó.

![History Mode](/images/2.Zero-ETLIntegration/142.png)

## Kết nối và chạy các cập nhật và xóa trên cơ sở dữ liệu nguồn

{{% notice info %}}  
Trong bước này, bạn sẽ trải nghiệm **Chế độ lịch sử** trong hành động. Tất cả các cập nhật/xóa thực hiện trên nguồn sẽ được đồng bộ gần như ngay lập tức, duy trì trạng thái của bảng thông qua 3 cột chế độ lịch sử sẽ được giải thích trong phần tiếp theo.
{{% /notice %}}

1. Trong bảng điều khiển **Aurora và RDS**
+ Điều hướng đến **Aurora PostgreSQL [Query Editor](https://console.aws.amazon.com/rds/home#query-editor:)**
+ Chọn cụm có tên **aurorapg-***.
+ Đối với tên người dùng cơ sở dữ liệu, chọn **Connect with a Secrets Manager ARN**.
+ Nhập giá trị ARN (Cách lấy ARN được giải thích trong bước 2). Nhập tên cơ sở dữ liệu là `postgres`.

![Create Stack](/images/2.Zero-ETLIntegration/112.png)

2. Kết nối với cơ sở dữ liệu Aurora PostgreSQL bằng ARN của Secrets Manager.
+ Bạn có thể lấy ARN từ tab **Outputs** của [cloudformation](https://console.aws.amazon.com/cloudformation/home).

![Create Stack](/images/2.Zero-ETLIntegration/113.png)

3. Sao chép câu lệnh sau vào trình soạn thảo và thực thi câu lệnh SQL.

```
update public.customer_info set job_title='History Mode Update Test' where customer_id=5001;
commit;
update public.customer_info set job_title='History Mode Update Test-Second Immediate Update' where customer_id=5001;
commit;
delete from public.customer_info where customer_id=5001;
commit;
```


![History Mode](/images/2.Zero-ETLIntegration/145.png)

![History Mode](/images/2.Zero-ETLIntegration/144.png)

4. Xác nhận rằng trạng thái **Resync initiated** (áp dụng cho giao dịch đầu tiên sau khi bật chế độ lịch sử) đã thay đổi thành **Synced** trong bảng điều khiển Redshift dưới tab **Table Statistics** của Zero-ETL integration.

![History Mode](/images/2.Zero-ETLIntegration/143.png)

## Xác thực trong đích Redshift

1. Điều hướng lại đến Amazon Redshift Query Editor v2
+ Thực thi các câu truy vấn xác thực và giám sát sau đây
+ Chọn cơ sở dữ liệu đích **aurorapostgres_zeroetl** đã được tạo từ integration hoặc sử dụng cú pháp ba phần <database.schema.table>.

```
select * from public.customer_info order by customer_id desc, _record_create_time desc, _record_delete_time desc;
```


![History Mode](/images/2.Zero-ETLIntegration/146.png)

Khi chế độ lịch sử được bật, các cột sau sẽ được thêm vào bảng đích của bạn để theo dõi các thay đổi trong nguồn. Mọi bản ghi nguồn bị xóa hoặc thay đổi sẽ tạo ra một bản ghi mới trong đích, dẫn đến số lượng bản ghi tổng cộng trong bảng đích tăng lên với nhiều phiên bản bản ghi. Các bản ghi sẽ không bị xóa khỏi bảng đích khi bị xóa hoặc sửa đổi trong nguồn. Bạn có thể quản lý các bảng đích bằng cách xóa các bản ghi không còn hoạt động.

| Tên Cột               | Kiểu Dữ Liệu | Mô Tả                                                                 |
|------------------------|--------------|------------------------------------------------------------------------|
| _record_is_active      | Boolean      | Chỉ ra liệu một bản ghi trong đích có đang hoạt động trong nguồn hay không. `True` chỉ ra rằng bản ghi đang hoạt động. |
| _record_create_time    | Timestamp    | Thời gian bắt đầu (UTC) khi bản ghi nguồn bắt đầu hoạt động.         |
| _record_delete_time    | Timestamp    | Thời gian kết thúc (UTC) khi bản ghi nguồn được cập nhật hoặc xóa.  |

{{% notice info %}}  
**Chúc mừng!** Bạn đã hoàn thành thành công một buổi demo về **CHẾ ĐỘ LỊCH SỬ** trong **Zero-ETL integrations**.
{{% /notice %}}
