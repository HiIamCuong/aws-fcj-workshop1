---
title : "History Mode"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 2.3.4 </b> "
---

## Overview
With **history mode**, you can configure your zero-ETL integrations to track every version (including updates and deletes) of your records in source tables, directly in **Amazon Redshift**. Using history mode, you can

+ Run a historical analysis,
+ Build look-back reports,
+ Perform trend analysis, and
+ Send incremental updates to downstream applications built on top of Amazon Redshift

## Turn on History Mode

1. Open and Connect Redshift Query Editor
+ Open **Amazon Redshift Query editor v2** (in case if not already open)
+ Connect to **Redshift Serverless datawarehouse destination** created as part of this workshop using **awsuser/Awsuser123** credentials as shown.

![History Mode](/images/2.Zero-ETLIntegration/34.png)

![History Mode](/images/2.Zero-ETLIntegration/38.png)

2. Execute the SQL below to check the count in target database **aurorapostgres_zeroetl** created from integration. 
+ Open another SQL editor in query editor v2 to see the newly created database in drop-down list. Alternatively, you can use [three-part notation](https://docs.aws.amazon.com/redshift/latest/dg/cross-database-overview.html) , i.e. **database_name.schema_name.object_name** to execute the query below using **aurorapostgres_zeroetl.public.<table_name>** instead.

```
ALTER DATABASE aurorapostgres_zeroetl INTEGRATION SET HISTORY_MODE = TRUE FOR TABLE public.customer_info;
```

![History Mode](/images/2.Zero-ETLIntegration/108.png)

+ This will change the table status to **Resync initiated** state. You can monitor this in Redshift console under Zero-ETL integration **Table Statistics** tab. The resync can take as much time as the seeding process (15-30 minutes) for that table.

![History Mode](/images/2.Zero-ETLIntegration/142.png)

## Connect and run updates and deletes on source database

{{% notice info %}}  
In this step, you experience **History mode** in action. All updates/deletes made to source would sync almost immediately maintaining the state of the table using 3 history mode columns explained in next section.
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
update public.customer_info set job_title='History Mode Update Test' where customer_id=5001;
commit;
update public.customer_info set job_title='History Mode Update Test-Second Immediate Update' where customer_id=5001;
commit;
delete from public.customer_info where customer_id=5001;
commit;
```

![History Mode](/images/2.Zero-ETLIntegration/145.png)

![History Mode](/images/2.Zero-ETLIntegration/144.png)

4. Verify the **Resync initiated** (applicable for the first transaction after setting history mode) changed to **Synced** in Redshift console under Zero-ETL integration **Table Statistics** tab.

![History Mode](/images/2.Zero-ETLIntegration/143.png)

## Validate in Redshift target

1. Navigate back to Amazon Redshift Query editor v2
+ Run following validation and monitoring queries
+ Choose target database **aurorapostgres_zeroetl** created from integration or use the 3 part notation <database.schema.table>.
```
select * from public.customer_info order by customer_id desc, _record_create_time desc, _record_delete_time desc;
```

![History Mode](/images/2.Zero-ETLIntegration/146.png)

When history mode is turned on, the following columns are added to your target table to keep track of changes in the source. Any source record that is deleted or changed creates a new record in the target, resulting in more total rows in the target with multiple record versions. Records are not deleted from the target table when deleted or modified in the source. You can manage target tables by deleting inactive records.

| Column Name           | Data Type | Description                                                                 |
|-----------------------|-----------|-----------------------------------------------------------------------------|
| _record_is_active     | Boolean   | Indicates if a record in the target is currently active in the source. `True` indicates the record is active. |
| _record_create_time   | Timestamp | Starting time (UTC) when the source record is active.                      |
| _record_delete_time   | Timestamp | Ending time (UTC) when the source record is updated or deleted.            |

{{% notice info %}}  
**Congratulations!** You have successfully completed a demonstration of **HISTORY MODE** in **Zero-ETL integrations**
{{% /notice %}}