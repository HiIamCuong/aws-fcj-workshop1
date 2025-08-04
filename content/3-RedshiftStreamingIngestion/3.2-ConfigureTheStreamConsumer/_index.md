---
title : "Configure the stream consumer"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 3.2 </b> "
---

## Create an external schema in Redshift

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

5. Within the Redshift Query Editor, run the following SQL command

```
CREATE EXTERNAL SCHEMA custpaytxn
FROM KINESIS
IAM_ROLE  default;
```
![ProducingData](/images/3.RedshiftStreamingIngestion/7.png)

This will create an external schema called **custpaytxn** which wil be used to map to the Kinesis Data Stream

You can verify if the schema created successfully by running the following query:

```
select * from SVV_EXTERNAL_SCHEMAS;
```

![ProducingData](/images/3.RedshiftStreamingIngestion/8.png)

6. Now you can create a materialized view to consume the stream data fro Kinesis. You will use a Redshift JSON function to the parse the data into individual columns

```
CREATE MATERIALIZED VIEW cust_payment_tx_stream
AUTO REFRESH YES
AS
SELECT approximate_arrival_timestamp ,
partition_key,
shard_id,
sequence_number,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TRANSACTION_ID')::bigint as TRANSACTION_ID,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_DATETIME')::character(50) as TX_DATETIME,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'CUSTOMER_ID')::int as CUSTOMER_ID,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TERMINAL_ID')::int as TERMINAL_ID,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_AMOUNT')::decimal(18,2) as TX_AMOUNT,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_TIME_SECONDS')::int as TX_TIME_SECONDS,
json_extract_path_text(from_varbyte(kinesis_data, 'utf-8'),'TX_TIME_DAYS')::int as TX_TIME_DAYS
FROM custpaytxn."cust-payment-txn-stream"
Where is_utf8(kinesis_data) AND can_json_parse(kinesis_data);
```

![ProducingData](/images/3.RedshiftStreamingIngestion/11.png)

You can also see that we have specified **AUTO REFRESH YES** in the code, this enables automatic refresh of the streaming ingestion view, which saves us time as we don't have to create additional data pipelines for refreshing

7. Confirm the data refresh every 1 minute from Kinesis to redshift materialized view (auto refreshed) by executing below SQL repeatedly in 1 minute intervals.

```
Select count(*) from cust_payment_tx_stream;
```

![ProducingData](/images/3.RedshiftStreamingIngestion/12.png)

8. Now lets query the streamed materialized view to see our sample data:

```
Select * from cust_payment_tx_stream limit 10;
```

![ProducingData](/images/3.RedshiftStreamingIngestion/13.png)

9. Lets also check how many records are in our streaming view now:

```
Select count(*) as stream_rec_count from cust_payment_tx_stream;
```

![ProducingData](/images/3.RedshiftStreamingIngestion/14.png)

{{% notice info %}}
Congratulations! You have now setup Redshift Streaming Ingestion using a Kinesis Data Stream and Redshift.
{{% /notice %}}