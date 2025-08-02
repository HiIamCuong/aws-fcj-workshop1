---
title : "Xác thực & Giám sát"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 2.1.3 </b> "
---

{{% notice info %}}  
Xác nhận rằng trạng thái tích hợp zero-etl của bạn đã chuyển từ **Creating** sang **Active**. Tích hợp hiện đã hoàn tất, và toàn bộ snapshot của nguồn sẽ được phản ánh nguyên trạng ở đích. Các thay đổi liên tục sẽ được đồng bộ gần như theo thời gian thực. Hãy giám sát và xác thực việc đồng bộ dữ liệu từ nguồn đến đích trong mô-đun tiếp theo.  
{{% /notice %}}

## Xác thực và giám sát đồng bộ dữ liệu giữa nguồn và đích
### Tạo cơ sở dữ liệu từ tích hợp trong Amazon Redshift
1. Để tạo cơ sở dữ liệu, hãy thực hiện các bước sau:
+ Trên trang quản lý Redshift Serverless, điều hướng đến namespace bắt đầu bằng **zero-etl-destination-***.
+ Chọn **Query data** trên giao diện namespace Redshift Serverless để mở [Query Editor v2](https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html). Nếu được yêu cầu, hãy cấu hình quyền truy cập.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

2. Trong trang cấu hình  
+ Chọn configure account

![Validations & Monitoring](/images/2.Zero-ETLIntegration/36.png)

3. Trong Redshift Query Editor V2  
+ Nhấp vào data warehouse có tiền tố tên **zero-etl-destination-***

![Validations & Monitoring](/images/2.Zero-ETLIntegration/37.png)

4. Kết nối tới Redshift Serverless  
+ Chọn **Other ways to connect**  
+ Chọn **Database user name and password**  
+ Nhập **User name**: `awsuser`  
+ Nhập **Password**: `Awsuser123`  
+ Nhấn **Create connection**

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

5. Tạo cơ sở dữ liệu  
+ Lấy **integration_id** từ bảng hệ thống **svv_integration**:

```
select integration_id from svv_integration;
```


+ Dùng kết quả từ bước trên để tạo cơ sở dữ liệu:

```
CREATE DATABASE rdsmysql_zeroetl FROM INTEGRATION '<result from above>';
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/43.png)

### Xác thực SEEDING và LỌC DỮ LIỆU tại đích

{{% notice info %}}  
Trong bước này, bạn xác thực rằng dữ liệu hiện tại từ nguồn đã được sao chép bằng cơ chế đồng bộ cấp lưu trữ (**SEEDING**). Sau đó, bạn xác thực rằng các bảng và schemas thiếu khóa chính đã được **lọc** như mong muốn.  
{{% /notice %}}

1. Mở và kết nối Redshift Query Editor  
+ Mở **Amazon Redshift Query Editor v2**  
+ Kết nối đến Redshift Serverless bằng tài khoản **awsuser / Awsuser123**

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Thực hiện truy vấn để kiểm tra số bản ghi trong database **rdsmysql_zeroetl**:

```
select '1. region' as tablename,count() from seedingdemo.region union
select '2. nation',count() from seedingdemo.nation union
select '3. supplier', count() from seedingdemo.supplier union
select '4. customer', count() from seedingdemo.customer union
select '5. orders',count() from seedingdemo.orders union
select '6. lineitem',count() from seedingdemo.lineitem order by 1;
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/45.png)

3. Kiểm tra xem các bảng đã bị lọc chưa:

```
select * from auroramysql_zeroetl.filter_missingpk.region;
select * from auroramysql_zeroetl.filter_missingpk.nation;
```
![Validations & Monitoring](/images/2.Zero-ETLIntegration/46.png)

## Kết nối, tạo và tải dữ liệu để kiểm tra CDC gần thời gian thực

{{% notice info %}}  
Trong bước này, bạn sẽ kiểm tra tính năng **CDC (Change Data Capture)** gần thời gian thực. Tất cả thay đổi tại nguồn sẽ được đồng bộ gần như ngay lập tức nếu tích hợp Zero-ETL đang ở trạng thái **Active**.  
{{% /notice %}}

1. Truy cập Aurora MySQL  
+ Truy cập **EC2 console**  
+ Chọn EC2 instance và nhấn **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. Trong trang kết nối  
+ Chọn **Session Manager**  
+ Nhấn **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ Bạn sẽ vào terminal CLI của EC2 instance

3. Lấy endpoint và port của Aurora MySQL  
+ Mở **Amazon RDS Console**  
+ Vào mục **Databases**  
+ Chọn database có tiền tố **zero-etl-lab-sourceamscluster-***  
+ Chọn **Role Writer**  
+ Sao chép endpoint

![Create Stack](/images/2.Zero-ETLIntegration/3.png)

4. Truy cập Aurora từ EC2 CLI:

```
mysql -h <aurora_mysql_writer_endpoint> -P 3306 -u awsuser -p
```


Password: `Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/5.png)

5. Sau khi kết nối, tạo bảng dữ liệu:

```
create database cdcdemo;

use cdcdemo;

create table part (
p_partkey int8 not null ,
p_name varchar(55) not null,
p_mfgr char(25) not null,
p_brand char(10) not null,
p_type varchar(25) not null,
p_size int4 not null,
PRIMARY KEY (P_PARTKEY)
);

create table partsupp (
ps_partkey int8 not null,
ps_suppkey int4 not null,
ps_availqty int4 not null,
ps_supplycost numeric(12,2) not null,
PRIMARY KEY (PS_PARTKEY, PS_SUPPKEY)
);
```


![Validations & Monitoring](/images/2.Zero-ETLIntegration/47.png)

6. Tải dữ liệu từ S3:

+ **us-east-1**:
```
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/part/' INTO TABLE part FIELDS TERMINATED BY '|';
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/partsupp/' INTO TABLE partsupp FIELDS TERMINATED BY '|';
```


+ **us-west-2**:
```
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/part/' INTO TABLE part FIELDS TERMINATED BY '|';
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/partsupp/' INTO TABLE partsupp FIELDS TERMINATED BY '|';
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/48.png)

### Xác thực đồng bộ CDC tại đích

1. Mở lại Redshift Query Editor  
+ Kết nối đến Redshift Serverless bằng tài khoản **awsuser / Awsuser123**

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Truy vấn kiểm tra lại dữ liệu trong **rdsmysql_zeroetl**:

```
select '1. region' as tablename,count() from seedingdemo.region union
select '2. nation',count() from seedingdemo.nation union
select '3. supplier', count() from seedingdemo.supplier union
select '4. customer', count() from seedingdemo.customer union
select '5. orders',count() from seedingdemo.orders union
select '6. lineitem',count() from seedingdemo.lineitem order by 1;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/45.png)

3. Truy vấn dữ liệu đồng bộ từ **cdcdemo**:

```
select '1. region' as tablename,count() from seedingdemo.region union
select '2. nation',count() from seedingdemo.nation union
select '3. supplier', count() from seedingdemo.supplier union
select '4. customer', count() from seedingdemo.customer union
select '5. orders',count() from seedingdemo.orders union
select '6. lineitem',count() from seedingdemo.lineitem union
select '7. part',count() from cdcdemo.part union
select '8. partsupp',count() from cdcdemo.partsupp order by 1;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/49.png)

## Giám sát

Bạn có thể theo dõi các chỉ số CloudWatch như số bảng được đồng bộ, độ trễ dữ liệu... trên Redshift console.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/50.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/51.png)
