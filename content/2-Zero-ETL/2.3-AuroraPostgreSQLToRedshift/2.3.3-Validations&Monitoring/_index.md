---
title : "Validations & Monitoring"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 2.3.3 </b> "
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

![Validations & Monitoring](/images/2.Zero-ETLIntegration/129.png)

+ Use the **integration_id** from the previous step to create a new database from the integration:
```
CREATE DATABASE aurorapostgres_zeroetl FROM INTEGRATION '<result from above>' DATABASE postgres;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/130.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/131.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/132.png)

### Validation of SEEDING and DATA FILTERTING in destination

{{% notice info %}}  
In this step, you validate that existing data in source is replicated by storage level sync, referred to as **SEEDING**. Then, you validate that tables and schemas (with missing primary key) we intended to **FILTER** were indeed filtered.
{{% /notice %}}

1. Open and Connect Redshift Query Editor
+ Open **Amazon Redshift Query editor v2** (in case if not already open)
+ Connect to **Redshift Serverless datawarehouse destination** created as part of this workshop using **awsuser/Awsuser123** credentials as shown.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Execute the SQL below to check the count in target database **aurorapostgres_zeroetl** created from integration. 
+ Open another SQL editor in query editor v2 to see the newly created database in drop-down list. 
+ Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **aurorapostgres_zeroetl.public.<table_name>** instead.

```
select count(*) from aurorapostgres_zeroetl.public.customer_info;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/133.png)

3. Validate **DATA FILTERING** prevented the schema and tables with missing primary keys that you filtered during integration creation. The SQL statements to select **customer_info_missing_pk** table records from **public** schema should fail with **ERROR: relation "public.customer_info_missing_pk" does not exist**. when run. The valid customer_info table with primary key was the only one synced.

```
select count(*) from aurorapostgres_zeroetl.public.customer_info_missing_pk;
select * from aurorapostgres_zeroetl.public.customer_info;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/134.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/135.png)

## Connect, Create and Load more data to experience near real-time CDC

{{% notice info %}}  
In this step, you experience **CDC (Change Data Capture)** happening in near real-time. All changes made to source would sync almost immediately, with an **Active** Zero-ETL integration.
{{% /notice %}}

1. In the **Aurora and RDS** dashboard
+ Navigate to the **Aurora PostgreSQL [Query Editor](https://console.aws.amazon.com/rds/home#query-editor:)**
+ Select the cluster named **aurorapg-***. 
+ For database username, choose **Connect with a Secrets Manager ARN**. 
+ Populate ARN value (How to retrieve explained in step 2). Enter the name of the database as `postgres`.

![Create Stack](/images/2.Zero-ETLIntegration/112.png)

2. Connect to the Aurora PostgreSQL database using the Secrets Manager ARN. 
+ You can get the ARN either from [cloudformation](https://console.aws.amazon.com/cloudformation/home) **Outputs** tab,

![Create Stack](/images/2.Zero-ETLIntegration/113.png)

3. Copy the following statement into the Editor and run the SQL statement.

```
create table public.newtable (
  new_key int8 not null ,
  new_name varchar(55) not null,
  PRIMARY KEY (new_key)
) ;

insert into public.newtable values(1,'dummy_record');

-- Append more rows to the seeding table
insert into public.customer_info values (5001, 'Dummy job title','dummyemail@example.com','Random Name','123-456-7890','Some City','Some State');
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/136.png)

### Validation of CDC data sync in destination

1. Open and Connect Redshift Query Editor
+ Open **Amazon Redshift Query editor v2** (in case if not already open)
+ Connect to **Redshift Serverless datawarehouse destination** created as part of this workshop using **awsuser/Awsuser123** credentials as shown.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/38.png)

2. Execute the SQL below to check the count in target database **aurorapostgres_zeroetl** created from integration (**Note** There should be 1 new records in **customer_info** table **customer_info** along with the **newtable** in **public** schema that you just loaded in aurora postgres; Will be in Amazon Redshift almost immediately). You may have to open another SQL editor in query editor v2 to see the newly created database in drop-down list. Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **aurorapostgres_zeroetl.public.<table_name>** instead.

```
select count(*) from aurorapostgres_zeroetl.public.customer_info;

select * from aurorapostgres_zeroetl.public.newtable;
```

![Validations & Monitoring](/images/2.Zero-ETLIntegration/137.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/138.png)

## Monitoring

You can check cloudwatch metrics like statistics of tables replicated, the lag in data freshness from your zero-ETL integration etc from Redshift console.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/139.png)

![Validations & Monitoring](/images/2.Zero-ETLIntegration/140.png)