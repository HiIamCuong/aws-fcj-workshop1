---
title : "Zero-ETL Integration Aurora MySQL to Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.1.1 </b> "
---

## Outline:
- Load your transactional database before creating Zero-ETL Integration. The first time sync of data is called SEEDING (storage level sync).

- Test ability to filter source tables or schemas using DATA FILTERING during creation of a Zero-ETL integration.

- For an Active Zero-ETL integration, experience near real-time CDC (change data capture) replication. Try HISTORY MODE.

## Connect to Aurora Mysql, Create and Load tables for SEEDING.

1. To connect to you Aurora Mysql instance 
+ Navigate to **EC2 console** 
+ Select your **EC2 instance** provisioned and click on **Connect**.

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. In the **Connect** page
+ Choose **Session manager**
+ Choose **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ You will go to CLI of EC2 instance

3. We need take Endpoint and Port of Aurora MySQL

+ Navigate go **Aurora and RDS Console**
+ Choose feature **Databases**
+ Choose database with name prefix **zero-etl-lab-sourceamscluster-***
+ Choose **Role Writer** below default choose
+ Copy **endpoint** of writer

![Create Stack](/images/2.Zero-ETLIntegration/3.png)

4. Go back CLI **EC2 instance**
+ Use this command to connect to your **Aurora MySQL** from **EC2 instance**

`mysql -h <aurora_mysql_writer_endpoint> -P 3306 -u awsuser -p `

+ Password to login

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/5.png)

5. Once connected with **Aurora MySQL instance**
+ Execute below DDLs to create tables for order-line dataset.

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

6. Load the data into tables created above from S3. (Just take 1 for your region)
+ **us-east-1** load commands:
```
--- For us-east-1 (N Virginia) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/supplier/' INTO TABLE supplier FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/customer/' INTO TABLE customer FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/orders/' INTO TABLE orders FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/lineitem/' INTO TABLE lineitem FIELDS TERMINATED BY '|';            
```
+ **us-west-2** load commands:
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

## Create and Load tables with missing primary key for demonstrating DATA FILTERING.

1. On your **Aurora Mysql instance**, Run below DDLs to create tables and schema that we would filter out during first time sync (or SEEDING). Note that we have deliberately removed the primary key from the DDL to make them candidates to filter out.
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
2. Load the data into tables created above from S3.(Just take 1 for your region)
+ **us-east-1** load commands:
```
--- For us-east-1 (N Virginia) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
```
+ **us-west-2** load commands:
```
--- For us-west-2 (Oregon) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/region/' INTO TABLE region FIELDS TERMINATED BY '|';          
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/nation/' INTO TABLE nation FIELDS TERMINATED BY '|';            
```
![Create Stack](/images/2.Zero-ETLIntegration/9.png)