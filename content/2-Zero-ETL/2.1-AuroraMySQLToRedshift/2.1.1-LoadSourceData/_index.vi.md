---
title : "Tải dữ liệu nguồn"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.1.1 </b> "
---

## Lời đầu
- Tải transaction database của bạn trước khi tạo Tích hợp Zero-ETL. Lần đồng bộ dữ liệu đầu tiên được gọi là SEEDING (đồng bộ cấp độ lưu trữ).

- Kiểm tra khả năng lọc các source table hoặc schemas bằng DATA FILTERING trong quá trình tạo tích hợp Zero-ETL.

- Đối với tích hợp Zero-ETL chủ động, trải nghiệm sao chép CDC (ghi lại dữ liệu thay đổi) gần như theo thời gian thực hãy thử HISTORY MODE.

## Kết nối đến Aurora MySQL, tạo và tải table để SEEDING

1. Để kết nối với phiên bản **Aurora Mysql** của bạn
+ Vào bảng điều khiển **EC2**
+ Chọn **EC2 instance** đã được tạo ở bước chuẩn bị và nhấp vào **Connect** .

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. Khi vào phần **Connect**
+ Chọn **Session manager**
+ Nhấn vào **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ Bạn sẽ vào được CLI của EC2 instance

3. Ta cần lấy Endpoint và Port của Aurora MySQL 

+ Vào bảng điều khiển **Aurora and RDS**
+ Nhấn vào mục **Databases**
+ Nhấn vào database có cấu trúc **zero-etl-lab-sourceamscluster-***
+ Nhấn chọn **Role Writer** bên dưới
+ Copy **endpoint** của writer

![Create Stack](/images/2.Zero-ETLIntegration/3.png)

4. Quay lại CLI của **EC2 instance**
+ Dùng lệnh sau để kết nối tới **Aurora MySQL** từ **EC2 instance** của bạn

`mysql -h <aurora_mysql_writer_endpoint> -P 3306 -u awsuser -p `

+ Nhập mật khẩu

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/5.png)

5. Khi đã kết nối với **Aurora MySQL instance**
+ Dùng lệnh SQL sau để lần lượt tạo các tables cho dataset

```
create database seedingdemo;

use seedingdemo;

create table region (
  r_regionkey int4 not null,
  r_name char(25) not null ,
  r_comment varchar(152) not null,
  Primary Key(R_REGIONKEY)                             
) ;

create table nation (
  n_nationkey int4 not null,
  n_name char(25) not null ,
  n_regionkey int4 not null,
  n_comment varchar(152) not null,
  Primary Key(N_NATIONKEY)                                
) ;

create table supplier (
  s_suppkey int4 not null,
  s_name char(25) not null,
  s_address varchar(40) not null,
  s_nationkey int4 not null,
  s_phone char(15) not null,
  s_acctbal numeric(12,2) not null,
  s_comment varchar(101) not null,
  Primary Key(S_SUPPKEY)
);

create table customer (
  c_custkey int8 not null ,
  c_nationkey int4 not null,
  c_acctbal numeric(12,2) not null,
  c_mktsegment char(10) not null,
  Primary Key(C_CUSTKEY)
);

create table orders (
  o_orderkey int8 not null,
  o_custkey int8 not null,
  o_orderstatus char(1) not null,
  o_orderpriority char(15) not null,
  o_shippriority int4 not null,
  o_clerk char(15) not null,
  Primary Key(O_ORDERKEY)
) ;

create table lineitem (
  l_orderkey int8 not null ,
  l_partkey int8 not null,
  l_suppkey int4 not null,
  l_linenumber int4 not null,
  l_returnflag char(1) not null,
  l_linestatus char(1) not null,
  l_shipmode char(10) not null,
  Primary Key(L_ORDERKEY, L_LINENUMBER)
)  ;
```
![Create Stack](/images/2.Zero-ETLIntegration/6.png)

6. Dùng lệnh sau để tải dữ liệu từ **S3** vào tables đã tạo ở trên (Tùy vào region mà bạn sử dụng chỉ lấy một lệnh)
+ Lệnh tải cho region **us-east-1**:
```
--- For us-east-1 (N Virginia) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/supplier/' INTO TABLE supplier FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/customer/' INTO TABLE customer FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/orders/' INTO TABLE orders FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/lineitem/' INTO TABLE lineitem FIELDS TERMINATED BY '|';            
```
+ Lệnh tải cho region **us-west-2**:
```
--- For us-west-2 (Oregon) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/supplier/' INTO TABLE supplier FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/customer/' INTO TABLE customer FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/orders/' INTO TABLE orders FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/lineitem/' INTO TABLE lineitem FIELDS TERMINATED BY '|';            
```
![Create Stack](/images/2.Zero-ETLIntegration/6.png)

## Tạo và tải các table thiếu khóa chính để minh họa DATA FILTERING

1. Tại **Aurora MySQL instance** của bạn. Chạy lệnh SQL dưới đây để tạo các table và schema mà chúng ta sẽ lọc ra trong lần đồng bộ hóa đầu tiên (SEEDING). Chúng ta cố tính xóa khóa chính khỏi SQL để chúng trở thành ứng cử viên cho việc lọc ra
```
create database filter_missingpk;

use filter_missingpk;

create table region (
  r_regionkey int4 not null,
  r_name char(25) not null ,
  r_comment varchar(152) not null                     
) ;

create table nation (
  n_nationkey int4 not null,
  n_name char(25) not null ,
  n_regionkey int4 not null,
  n_comment varchar(152) not null                        
) ;

```
![Create Stack](/images/2.Zero-ETLIntegration/8.png)
2. Tạo dữ liệu vào các bảng đã tạo ở trên từ **S3**(Tùy vào region mà bạn sử dụng chỉ lấy một lệnh)
+ Lệnh tải cho region **us-east-1**:
```
--- For us-east-1 (N Virginia) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
```
+ Lệnh tải cho region **us-west-2**:
```
--- For us-west-2 (Oregon) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
```
![Create Stack](/images/2.Zero-ETLIntegration/9.png)