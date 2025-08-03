---
title : "Load Source Data"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.2.1 </b> "
---

## Outline:
- Load your transactional database before creating Zero-ETL Integration. The first time sync of data is called SEEDING (storage level sync).

- Test ability to filter source tables or schemas using DATA FILTERING during creation of a Zero-ETL integration. 

- For an Active Zero-ETL integration, experience near real-time CDC (change data capture) replication. Optionally, try HISTORY MODE.

## Connect to RDS for Mysql, Create and Load tables for SEEDING.

1. To connect to your RDS Mysql instance 
+ Navigate to **EC2 console** 
+ Select your **EC2 instance** provisioned and click on **Connect**.

![Create Stack](/images/2.Zero-ETLIntegration/1.png)

2. In the **Connect** page
+ Choose **Session manager**
+ Choose **Connect**

![Create Stack](/images/2.Zero-ETLIntegration/2.png)

+ You will go to CLI of EC2 instance

3. We need take Endpoint and Port of RDS for MySQL

+ Navigate go **Aurora and RDS Console**
+ Choose feature **Databases**
+ Choose database with name prefix **zero-etl-lab-sourcerdsmscluster-***
+ Choose **Role Writer** below default choose
+ Copy **endpoint** of writer

![Create Stack](/images/2.Zero-ETLIntegration/57.png)

4. Go back CLI **EC2 instance**
+ Use this command to connect to your **Aurora MySQL** from **EC2 instance**

`mysql -h rds_mysql_instance_endpoint -P 6612 -u awsuser -p `

+ Password to login

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/58.png)

5. Once connected with **RDS MySQL instance**
+ Execute below DDLs to create tables for order-line dataset.

```
create database seedingdemo;

use seedingdemo;

CREATE TABLE customer_info
(
customer_id   BIGINT primary key,
job_title     VARCHAR(500),
email_address VARCHAR(100),
full_name     VARCHAR(200),
phone_number  VARCHAR(20),
city          VARCHAR(50),
state         VARCHAR(50)
);
```

![Create Stack](/images/2.Zero-ETLIntegration/59.png)

6. Load the data into tables created above
```
LOAD DATA LOCAL INFILE '/zetldata/customer_info.csv' INTO TABLE customer_info FIELDS TERMINATED BY ',' IGNORE 1 ROWS;
```
![Create Stack](/images/2.Zero-ETLIntegration/60.png)

## Create and Load tables with missing primary key for demonstrating DATA FILTERING.

1. On your **RDS for Mysql instance**, Run below DDLs to create tables and schema that we would **filter out** during first time sync (or SEEDING). Note that we have deliberately removed the primary key from the DDL to make them candidates to filter out.
```
create database filter_missingpk;

use filter_missingpk;

CREATE TABLE customer_info
(
customer_id   BIGINT,
job_title     VARCHAR(500),
email_address VARCHAR(100),
full_name     VARCHAR(200),
phone_number  VARCHAR(20),
city          VARCHAR(50),
state         VARCHAR(50)
);
```

![Create Stack](/images/2.Zero-ETLIntegration/61.png)

2. Load the data into primary key less table created above.

![Create Stack](/images/2.Zero-ETLIntegration/62.png)