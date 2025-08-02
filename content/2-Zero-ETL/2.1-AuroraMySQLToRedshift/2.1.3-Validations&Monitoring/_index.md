---
title : "Validations & Monitoring"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 2.1.3 </b> "
---

{{% notice info %}}  
Confirm that your zero-etl integration status has changed from **Creating** to **Active**. The integration is now complete, and an entire snapshot of the source will reflect as is in the destination. Ongoing changes will be synced in near-real time. Monitor and Validate your source data sync to destination in next module.
{{% /notice %}}

## Validate and monitor the data sync between source and destination
### Create a database from the integration in Amazon Redshift
1. To create your database, complete the following steps:
+ On the Redshift Serverless dashboard, navigate to the namespace starting with **zero-etl-destination-*** namespace.
+ Choose **Query data** on redshift serverless namespace console to open [Query Editor v2](https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html) . Proceed with *configure* if prompted for permissions.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

2. In the configuration page
+ Choose configure account

![Validations & Monitoring](/images/2.Zero-ETLIntegration/36.png)

3. In Redshift Query Editor V2
+ Click on datawarehouse with name prefix **zero-etl-destination-***

![Validations & Monitoring](/images/2.Zero-ETLIntegration/37.png)

4. Connect to the preview Redshift Serverless data warehouse
+ Choose **Other ways to connect**
+ Choose **Database user name and password**
+ Fill **User name**: `awsuser`
+ Fill **Password**: `Awsuser123`
+ Choosing **Create connection**.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

5. Tạo database
+ Obtain the **integration_id** from the **svv_integration** system table:

```
-- copy this result, use in the next sql
select integration_id from svv_integration; 
```

+ Use the **integration_id** from the previous step to create a new database from the integration:
```
CREATE DATABASE rdsmysql_zeroetl FROM INTEGRATION '<result from above>';
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/43.png)

### Validation of SEEDING and DATA FILTERTING in destination

{{% notice info %}}  
In this step, you validate that existing data in source is replicated by storage level sync, referred to as **SEEDING**. Then, you validate that tables and schemas (with missing primary key) we intended to **FILTER** were indeed filtered.
{{% /notice %}}

1. Open and Connect Redshift Query Editor
+ Open **Amazon Redshift Query editor v2** (in case if not already open)
+ Connect to **Redshift Serverless datawarehouse destination** created as part of this workshop using **awsuser/Awsuser123** credentials as shown.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Execute the SQL below to check the count in target database **rdsmysql_zeroetl** created from integration. 
+ Open another SQL editor in query editor v2 to see the newly created database in drop-down list. 
+ Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **rdsmysql_zeroetl.seedingdemo.<table_name>** instead.

```
select '1. region' as tablename,count(*) from seedingdemo.region union
select '2. nation',count(*) from seedingdemo.nation union
select '3. supplier', count(*) from seedingdemo.supplier union
select '4. customer', count(*) from seedingdemo.customer union
select '5. orders',count(*) from seedingdemo.orders union
select '6. lineitem',count(*) from seedingdemo.lineitem order by 1;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/45.png)

3. Validate DATA FILTERING prevented the schema and tables with missing primary keys that you filtered during integration creation.
+ The SQL statements to select region and nation table records from filter_missingpk schema should fail with Table/Schema does not exist when run.

```
select * from auroramysql_zeroetl.filter_missingpk.region;
select * from auroramysql_zeroetl.filter_missingpk.nation;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/46.png)

## Connect, Create and Load more data to experience near real-time CDC

{{% notice info %}}  
In this step, you experience **CDC (Change Data Capture)** happening in near real-time. All changes made to source would sync almost immediately, with an **Active** Zero-ETL integration.
{{% /notice %}}

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

5. Once connected to your Aurora Mysql instance
+ Execute below DDLs to create tables for order-line dataset.

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
) ;

create table partsupp (
  ps_partkey int8 not null,
  ps_suppkey int4 not null,
  ps_availqty int4 not null,
  ps_supplycost numeric(12,2) not null,
  Primary Key(PS_PARTKEY, PS_SUPPKEY)
) ;

```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/47.png)

6. Load the data into tables created above from S3. (Just take 1 for your region)
+ **us-east-1** load commands:
```
--- For us-east-1 (N Virginia) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/part/' INTO TABLE part FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-demos/ri2023/ant307/data/order-line/partsupp/' INTO TABLE partsupp FIELDS TERMINATED BY '|';            

```

+ **us-west-2** load commands:
```
--- For us-west-2 (Oregon) region, please use below load scripts:
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/part/' INTO TABLE part FIELDS TERMINATED BY '|';            
LOAD DATA FROM S3 PREFIX 's3://redshift-immersionday-labs/ri2023/ant307/data/order-line/partsupp/' INTO TABLE partsupp FIELDS TERMINATED BY '|';        
         
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/48.png)

### Validation of CDC data sync in destination

1. Open and Connect Redshift Query Editor
+ Open **Amazon Redshift Query editor v2** (in case if not already open)
+ Connect to **Redshift Serverless datawarehouse destination** created as part of this workshop using **awsuser/Awsuser123** credentials as shown.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Execute the SQL below to check the count in target database **rdsmysql_zeroetl** created from integration. 
+ Open another SQL editor in query editor v2 to see the newly created database in drop-down list. 
+ Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **rdsmysql_zeroetl.seedingdemo.<table_name>** instead.

```
select '1. region' as tablename,count(*) from seedingdemo.region union
select '2. nation',count(*) from seedingdemo.nation union
select '3. supplier', count(*) from seedingdemo.supplier union
select '4. customer', count(*) from seedingdemo.customer union
select '5. orders',count(*) from seedingdemo.orders union
select '6. lineitem',count(*) from seedingdemo.lineitem order by 1;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/45.png)

3. Execute the SQL below to check the count in target database **auroramysql_zeroetl** created from integration (**Note** the last 2 tables from **cdcdemo** schema that you just loaded in Aurora MySQL; Will be in Amazon Redshift almost immediately). 
+ Open another SQL editor in query editor v2 to see the newly created database in drop-down list. Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **auroramysql_zeroetl.demodb.<table_name>** instead.
```
select '1. region' as tablename,count(*) from seedingdemo.region union
select '2. nation',count(*) from seedingdemo.nation union
select '3. supplier', count(*) from seedingdemo.supplier union
select '4. customer', count(*) from seedingdemo.customer union
select '5. orders',count(*) from seedingdemo.orders union
select '6. lineitem',count(*) from seedingdemo.lineitem union
select '7. part',count(*) from cdcdemo.part union
select '8. partsupp',count(*) from cdcdemo.partsupp order by 1;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/49.png)

## Monitoring

You can check cloudwatch metrics like statistics of tables replicated, the lag in data freshness from your zero-ETL integration etc from Redshift console.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/50.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/51.png)