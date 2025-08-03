---
title : "Xác thực & Giám sát"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 2.1.3 </b> "
---

{{% notice info %}}  
Xác nhận rằng trạng thái của Zero-ETL integration đã thay đổi từ **Creating** sang **Active**. Việc tích hợp đã hoàn tất, và một bức tranh toàn bộ của nguồn dữ liệu sẽ được phản ánh ngay lập tức tại đích. Những thay đổi liên tục sẽ được đồng bộ gần như thời gian thực. Hãy theo dõi và xác minh quá trình đồng bộ dữ liệu từ nguồn đến đích trong module tiếp theo.
{{% /notice %}}

## Xác minh và giám sát quá trình đồng bộ dữ liệu giữa nguồn và đích
### Tạo cơ sở dữ liệu từ integration trong Amazon Redshift
1. Để tạo cơ sở dữ liệu của bạn, thực hiện các bước sau:
+ Trên bảng điều khiển **Redshift Serverless**, điều hướng đến không gian tên bắt đầu bằng **zero-etl-destination-***.
+ Chọn **Query data** trên bảng điều khiển không gian tên Redshift Serverless để mở [Query Editor v2](https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html). Tiến hành *configure* nếu được yêu cầu cấp quyền.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

2. Trên trang cấu hình
+ Chọn **configure account**

![Validations & Monitoring](/images/2.Zero-ETLIntegration/36.png)

3. Trong Redshift Query Editor V2
+ Nhấn vào data warehouse có tiền tố tên **zero-etl-destination-***

![Validations & Monitoring](/images/2.Zero-ETLIntegration/37.png)

4. Kết nối đến Redshift Serverless data warehouse preview
+ Chọn **Other ways to connect**
+ Chọn **Database user name and password**
+ Điền **User name**: `awsuser`
+ Điền **Password**: `Awsuser123`
+ Chọn **Create connection**.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

5. Tạo cơ sở dữ liệu
+ Lấy **integration_id** từ bảng hệ thống **svv_integration**:

```
-- sao chép kết quả này, sử dụng trong câu SQL tiếp theo
select integration_id from svv_integration;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/98.png)

+ Sử dụng **integration_id** từ bước trước để tạo một cơ sở dữ liệu mới từ integration:

```
CREATE DATABASE rdsmysql_zeroetl FROM INTEGRATION '<kết quả từ trên>';
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/99.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/100.png)

### Xác minh SEEDING và DATA FILTERING tại đích

{{% notice info %}}  
Trong bước này, bạn xác minh rằng dữ liệu hiện có trong nguồn đã được sao chép qua đồng bộ cấp lưu trữ, được gọi là **SEEDING**. Sau đó, bạn xác minh rằng các bảng và lược đồ (thiếu khóa chính) mà bạn đã **FILTER** thực sự đã bị lọc.
{{% /notice %}}

1. Mở và Kết nối với Redshift Query Editor
+ Mở **Amazon Redshift Query editor v2** (nếu chưa mở)
+ Kết nối với **Redshift Serverless data warehouse destination** được tạo trong workshop này bằng tài khoản **awsuser/Awsuser123** như đã chỉ định.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi câu SQL dưới đây để kiểm tra số lượng trong cơ sở dữ liệu đích **rdsmysql_zeroetl** được tạo từ integration. 
+ Mở một trình soạn thảo SQL khác trong query editor v2 để xem cơ sở dữ liệu mới tạo trong danh sách thả xuống. 
+ Ngoài ra, bạn có thể sử dụng [ký hiệu ba phần](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html), tức là **database_name.schema_name.object_name** để thực thi truy vấn dưới đây sử dụng **rdsmysql_zeroetl.seedingdemo.<table_name>** thay thế.

```
select count(*) from rdsmysql_zeroetl.seedingdemo.customer_info;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/101.png)

3. Xác minh **DATA FILTERING** đã ngăn chặn các lược đồ và bảng thiếu khóa chính mà bạn đã lọc trong quá trình tạo integration. Các câu lệnh SQL để chọn bản ghi bảng **customer_info** từ lược đồ **filter_missingpk** sẽ gặp lỗi với thông báo **ERROR: Could not find parent table for alias "rdsmysql_zeroetl.filter_missingpk.customer_info"** khi chạy.

```
select count(*) from rdsmysql_zeroetl.filter_missingpk.customer_info;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/102.png)

## Kết nối, tạo và tải thêm dữ liệu để trải nghiệm CDC gần thời gian thực

{{% notice info %}}  
Trong bước này, bạn sẽ trải nghiệm **CDC (Change Data Capture)** xảy ra gần thời gian thực. Tất cả các thay đổi được thực hiện trên nguồn sẽ được đồng bộ gần như ngay lập tức, với Zero-ETL integration **Active**.
{{% /notice %}}

1. Để kết nối với Aurora MySQL của bạn 
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
+ Sử dụng lệnh này để kết nối tới **Aurora MySQL** từ **EC2 instance**

`mysql -h rds_mysql_instance_endpoint -P 6612 -u awsuser -p `

+ Mật khẩu để đăng nhập

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/5.png)

5. Khi đã kết nối với **RDS MySQL instance**
+ Thực thi các lệnh DDL dưới đây để tạo bảng và thêm một số bản ghi CDC.

```
create database cdcdemo;

use cdcdemo;

create table newtable (
new_key int8 not null ,
new_name varchar(55) not null,
PRIMARY KEY (new_key)
) ;

insert into newtable values(1,'dummy_record');

-- Thêm các dòng mới vào bảng seeding
use seedingdemo;

LOAD DATA LOCAL INFILE '/zetldata/customer_info_incr.csv' INTO TABLE customer_info FIELDS TERMINATED BY ',' IGNORE 1 ROWS;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/103.png)

### Xác minh đồng bộ dữ liệu CDC tại đích

1. Mở và Kết nối với Redshift Query Editor
+ Mở **Amazon Redshift Query editor v2** (nếu chưa mở)
+ Kết nối với **Redshift Serverless data warehouse destination** được tạo trong workshop này bằng tài khoản **awsuser/Awsuser123** như đã chỉ định.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Thực thi câu SQL dưới đây để kiểm tra số lượng trong cơ sở dữ liệu đích **rdsmysql_zeroetl** được tạo từ integration (**Lưu ý** sẽ có 2 bản ghi mới trong bảng **customer_info** lược đồ **seedingdemo** cùng với **newtable** trong lược đồ **cdcdemo** mà bạn vừa tải vào RDS cho MySQL; sẽ có mặt trong Amazon Redshift gần như ngay lập tức). Bạn có thể phải mở một trình soạn thảo SQL khác trong query editor v2 để xem cơ sở dữ liệu mới tạo trong danh sách thả xuống. Ngoài ra, bạn có thể sử dụng [ký hiệu ba phần](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html), tức là **database_name.schema_name.object_name** để thực thi truy vấn dưới đây sử dụng **rdsmysql_zeroetl.cdcdemo.<table_name>** thay thế.

```
select count(*) from rdsmysql_zeroetl.seedingdemo.customer_info;

select * from rdsmysql_zeroetl.cdcdemo.newtable;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/104.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/105.png)

## Giám sát

Bạn có thể kiểm tra các chỉ số trong CloudWatch như thống kê các bảng đã được sao chép, độ trễ trong độ tươi của dữ liệu từ Zero-ETL integration của bạn, v.v., từ bảng điều khiển Redshift.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/106.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/107.png)
