---
title : "Xác thực & Giám sát"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 2.3.3 </b> "
---

{{% notice info %}}  
Xác nhận rằng trạng thái của zero-etl integration đã chuyển từ **Creating** sang **Active**. Quá trình tích hợp đã hoàn tất, và một bản sao toàn bộ dữ liệu từ nguồn sẽ được phản ánh chính xác tại đích. Các thay đổi tiếp theo sẽ được đồng bộ gần như ngay lập tức. Hãy theo dõi và xác thực việc đồng bộ dữ liệu từ nguồn sang đích trong module tiếp theo.
{{% /notice %}}

## Xác thực và giám sát việc đồng bộ dữ liệu giữa nguồn và đích

### Tạo cơ sở dữ liệu từ integration trong Amazon Redshift

1. Để tạo cơ sở dữ liệu của bạn, thực hiện các bước sau:
+ Trên bảng điều khiển Redshift Serverless, điều hướng đến không gian tên bắt đầu bằng **zero-etl-destination-***.
+ Chọn **Query data** trên bảng điều khiển không gian tên Redshift Serverless để mở [Query Editor v2](https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html). Tiến hành cấu hình nếu có yêu cầu quyền truy cập.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

2. Trong trang cấu hình
+ Chọn cấu hình tài khoản

![Validations & Monitoring](/images/2.Zero-ETLIntegration/36.png)

3. Trong Redshift Query Editor V2
+ Nhấp vào kho dữ liệu có tiền tố tên **zero-etl-destination-***

![Validations & Monitoring](/images/2.Zero-ETLIntegration/37.png)

4. Kết nối với kho dữ liệu Redshift Serverless để xem trước
+ Chọn **Other ways to connect**
+ Chọn **Database user name and password**
+ Điền **User name**: `awsuser`
+ Điền **Password**: `Awsuser123`
+ Chọn **Create connection**.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

5. Tạo cơ sở dữ liệu
+ Lấy **integration_id** từ bảng hệ thống **svv_integration**:

```
-- sao chép kết quả này, sử dụng trong câu lệnh sql tiếp theo
select integration_id from svv_integration;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/129.png)

+ Sử dụng **integration_id** từ bước trước để tạo cơ sở dữ liệu mới từ integration:

```
CREATE DATABASE aurorapostgres_zeroetl FROM INTEGRATION '<result from above>' DATABASE postgres;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/130.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/131.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/132.png)

### Xác thực SEEDING và DATA FILTERING trong đích

{{% notice info %}}  
Trong bước này, bạn xác thực rằng dữ liệu hiện tại trong nguồn đã được đồng bộ qua **SEEDING** (đồng bộ dữ liệu ở cấp độ lưu trữ). Sau đó, bạn xác thực rằng các bảng và schema (thiếu khóa chính) mà chúng ta đã chọn **FILTER** thực sự đã bị lọc.
{{% /notice %}}

1. Mở và kết nối với Redshift Query Editor
+ Mở **Amazon Redshift Query editor v2** (nếu chưa mở)
+ Kết nối với **Redshift Serverless datawarehouse destination** đã được tạo trong workshop này bằng thông tin tài khoản **awsuser/Awsuser123** như đã chỉ ra.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi câu lệnh SQL dưới đây để kiểm tra số lượng trong cơ sở dữ liệu đích **aurorapostgres_zeroetl** được tạo từ integration.
+ Mở một trình soạn thảo SQL khác trong query editor v2 để xem cơ sở dữ liệu mới tạo trong danh sách thả xuống.
+ Ngoài ra, bạn có thể sử dụng [cú pháp ba phần](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html), tức là **database_name.schema_name.object_name** để thực thi câu truy vấn dưới đây bằng cách sử dụng **aurorapostgres_zeroetl.public.<table_name>**.

```
select count(*) from aurorapostgres_zeroetl.public.customer_info;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/133.png)

3. Xác thực rằng **DATA FILTERING** đã ngăn không cho schema và bảng thiếu khóa chính mà bạn đã lọc trong quá trình tạo integration. Câu lệnh SQL để chọn các bản ghi của bảng **customer_info_missing_pk** từ schema **public** sẽ bị lỗi với thông báo **ERROR: relation "public.customer_info_missing_pk" does not exist** khi chạy. Bảng **customer_info** hợp lệ với khóa chính là bảng duy nhất được đồng bộ.

```
select count(*) from aurorapostgres_zeroetl.public.customer_info_missing_pk;
select * from aurorapostgres_zeroetl.public.customer_info;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/134.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/135.png)

## Kết nối, tạo và tải thêm dữ liệu để trải nghiệm CDC gần thời gian thực

{{% notice info %}}  
Trong bước này, bạn sẽ trải nghiệm **CDC (Change Data Capture)** diễn ra gần như ngay lập tức. Mọi thay đổi trong nguồn sẽ được đồng bộ gần như ngay lập tức với Zero-ETL integration đang ở trạng thái **Active**.
{{% /notice %}}

1. Trên bảng điều khiển **Aurora and RDS**
+ Điều hướng đến **Aurora PostgreSQL [Query Editor](https://console.aws.amazon.com/rds/home#query-editor:)**
+ Chọn cụm có tên **aurorapg-***. 
+ Đối với tên người dùng cơ sở dữ liệu, chọn **Connect with a Secrets Manager ARN**.
+ Nhập giá trị ARN (Cách lấy ARN được giải thích trong bước 2). Nhập tên cơ sở dữ liệu là `postgres`.

![Create Stack](/images/2.Zero-ETLIntegration/112.png)

2. Kết nối với cơ sở dữ liệu Aurora PostgreSQL sử dụng ARN của Secrets Manager. 
+ Bạn có thể lấy ARN từ tab **Outputs** của [cloudformation](https://console.aws.amazon.com/cloudformation/home).

![Create Stack](/images/2.Zero-ETLIntegration/113.png)

3. Sao chép câu lệnh dưới đây vào Editor và thực thi câu lệnh SQL.

```
create table public.newtable (
new_key int8 not null ,
new_name varchar(55) not null,
PRIMARY KEY (new_key)
) ;

insert into public.newtable values(1,'dummy_record');

-- Thêm nhiều hàng vào bảng seeding
insert into public.customer_info values (5001, 'Dummy job title','dummyemail@example.com','Random Name','123-456-7890','Some City','Some State');
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/136.png)

### Xác thực việc đồng bộ dữ liệu CDC tại đích

1. Mở và kết nối với Redshift Query Editor
+ Mở **Amazon Redshift Query editor v2** (nếu chưa mở)
+ Kết nối với **Redshift Serverless datawarehouse destination** đã tạo trong workshop này bằng thông tin tài khoản **awsuser/Awsuser123** như đã chỉ ra.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi câu lệnh SQL dưới đây để kiểm tra số lượng trong cơ sở dữ liệu đích **aurorapostgres_zeroetl** được tạo từ integration (**Lưu ý**: Sẽ có 1 bản ghi mới trong bảng **customer_info** và **newtable** trong schema **public** mà bạn vừa tải vào aurora postgres; Sẽ có mặt trên Amazon Redshift gần như ngay lập tức). Bạn có thể phải mở một trình soạn thảo SQL khác trong query editor v2 để xem cơ sở dữ liệu mới tạo trong danh sách thả xuống. Ngoài ra, bạn có thể sử dụng [cú pháp ba phần](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html), tức là **database_name.schema_name.object_name** để thực thi câu truy vấn dưới đây bằng cách sử dụng **aurorapostgres_zeroetl.public.<table_name>**.

```
select count(*) from aurorapostgres_zeroetl.public.customer_info;

select * from aurorapostgres_zeroetl.public.newtable;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/137.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/138.png)

## Giám sát

Bạn có thể kiểm tra các chỉ số CloudWatch như thống kê bảng đã được sao chép, độ trễ trong sự tươi mới dữ liệu từ Zero-ETL integration của bạn từ bảng điều khiển Redshift.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/139.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/140.png)
