---
title : "Validations & Monitoring"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 2.2.3 </b> "
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

![Validations & Monitoring](/images/2.Zero-ETLIntegration/98.png)

+ Use the **integration_id** from the previous step to create a new database from the integration:
```
CREATE DATABASE rdsmysql_zeroetl FROM INTEGRATION '<result from above>';
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/99.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/100.png)

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
select count(*) from rdsmysql_zeroetl.seedingdemo.customer_info;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/101.png)

3. Validate **DATA FILTERING** prevented the schema and tables with missing primary keys that you filtered during integration creation. The SQL statements to select **customer_info** table records from **filter_missingpk** schema should fail with **ERROR: Could not find parent table for alias "rdsmysql_zeroetl.filter_missingpk.customer_info"**. when run.

```
select count(*) from rdsmysql_zeroetl.filter_missingpk.customer_info;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/102.png)

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
+ Choose database with name prefix **zero-etl-lab-sourcerdsmscluster-***
+ Choose **Role Writer** below default choose
+ Copy **endpoint** of writer

![Create Stack](/images/2.Zero-ETLIntegration/57.png)

4. Go back CLI **EC2 instance**
+ Use this command to connect to your **Aurora MySQL** from **EC2 instance**

`mysql -h rds_mysql_instance_endpoint -P 6612 -u awsuser -p `

+ Password to login

`Awsuser123`

![Create Stack](/images/2.Zero-ETLIntegration/5.png)

5. Once connected to your RDS for Mysql instance
+ Execute below DDLs to create tables and add some CDC records.

```
create database cdcdemo;

use cdcdemo;

create table newtable (
  new_key int8 not null ,
  new_name varchar(55) not null,
  PRIMARY KEY (new_key)
) ;

insert into newtable values(1,'dummy_record');

-- Append more rows to the seeding table
use seedingdemo;

LOAD DATA LOCAL INFILE '/zetldata/customer_info_incr.csv' INTO TABLE customer_info FIELDS TERMINATED BY ',' IGNORE 1 ROWS;

```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/103.png)

### Validation of CDC data sync in destination

1. Open and Connect Redshift Query Editor
+ Open **Amazon Redshift Query editor v2** (in case if not already open)
+ Connect to **Redshift Serverless datawarehouse destination** created as part of this workshop using **awsuser/Awsuser123** credentials as shown.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Execute the SQL below to check the count in target database **rdsmysql_zeroetl** created from integration (**Note** There should be 2 new records in **seedingdemo schema** table **customer_info** along with the **newtable** in cdcdemo schema that you just loaded in RDS for MySQL; Will be in Amazon Redshift almost immediately). You may have to open another SQL editor in query editor v2 to see the newly created database in drop-down list. Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **rdsmysql_zeroetl.cdcdemo.<table_name>** instead.

```
select count(*) from rdsmysql_zeroetl.seedingdemo.customer_info;

select * from rdsmysql_zeroetl.cdcdemo.newtable;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/104.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/105.png)

## Monitoring

You can check cloudwatch metrics like statistics of tables replicated, the lag in data freshness from your zero-ETL integration etc from Redshift console.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/106.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/107.png)