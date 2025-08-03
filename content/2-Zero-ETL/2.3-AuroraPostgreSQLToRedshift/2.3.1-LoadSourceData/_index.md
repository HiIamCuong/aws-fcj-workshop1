---
title : "Load Source Data"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 2.3.1 </b> "
---

## Outline:
- Load your transactional database before creating Zero-ETL Integration. The first time sync of data is called SEEDING (storage level sync).

- Test ability to filter source tables or schemas using DATA FILTERING during creation of a Zero-ETL integration.

- For an Active Zero-ETL integration, experience near real-time CDC (change data capture) replication. Optionally, try HISTORY MODE.

## Connect to Aurora PostgreSQL, verify SEEDING table present.

View the data on the customer_info table in the Aurora PostgreSQL database.

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
select * from public.customer_info;
```

![Create Stack](/images/2.Zero-ETLIntegration/114.png)

![Create Stack](/images/2.Zero-ETLIntegration/115.png)

{{% notice info %}}
Creation of the source data that will be replicated using Zero-ETL integration is complete. In following section, we create some useless data that we will filter during creation of Zero-ETL integration. Typical example of such a dataset is transactional table with missing primary keys.
{{% /notice %}}

## Create and Load tables with missing primary key for demonstrating DATA FILTERING.

1. On your Aurora PostgreSQL [Query Editor](https://console.aws.amazon.com/rds/home#query-editor:) 
+ Run below DDLs to create tables and schema that we would **filter out** during first time sync (or SEEDING). Note that we have deliberately removed the primary key from the DDL to make them candidates to filter out.
```
CREATE TABLE public.customer_info_missing_pk
(
customer_id   BIGINT,
job_title     VARCHAR(500),
email_address VARCHAR(100),
full_name     VARCHAR(200),
phone_number  VARCHAR(20),
city          VARCHAR(50),
state         VARCHAR(50)
);

insert into public.customer_info_missing_pk values (1, 'Dummy job title','dummyemail@example.com','Random Name','123-456-7890','Some City','Some State');
```

![Create Stack](/images/2.Zero-ETLIntegration/116.png)