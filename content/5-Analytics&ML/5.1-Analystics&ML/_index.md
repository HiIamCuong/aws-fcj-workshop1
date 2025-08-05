---
title : "Analyze all your data"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

{{% notice info %}}
Ensure your [Zero-ETL Integration](2-Zero-ETL) is Active and validate that the data from Aurora MySQL cluster is synced to Amazon Redshift.
{{% /notice %}}

## Bring it all together

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

5. Let's run an analytic query to analyze all the source datasets instantly. Great ideas should not have to wait. See how ExampleCorp will run a Pop-Up campaign right away by analyzing all its data.

```
---- ExampleCorp would like to run a targeted promotional campaign for it's Top100 urgent priority customers with highest spend.

SELECT c.full_name,
       c.email_address,
       o.o_orderpriority,
       Sum(h.tx_amount) cust_past_spend,
       Sum(p.tx_amount) cust_curr_spend
FROM   auroramysql_zeroetl.seedingdemo.orders o        ------ Zero-ETL Integration Source: Aurora MySQL
--       INNER JOIN postgres.customer_info c           ------ Federated Query Source: Aurora Postgres (This data is in 3 systems; pick from either)
--       INNER JOIN rdsmysql_zeroetl.seedingdemo.customer_info c   ------ Zero-ETL Integration Source: RDS for MySQL
       INNER JOIN aurorapostgres_zeroetl.public.customer_info c   ------ Zero-ETL Integration Source: Aurora PostgreSQL
               ON o.o_custkey = c.customer_id
       INNER JOIN (SELECT customer_id,
                          Sum(tx_amount) tx_amount
                   FROM   ant307_external.cust_payment_tx_history
                   GROUP  BY 1) h                       ------ Federated Query Source: Amazon S3
               ON o.o_custkey = h.customer_id
       INNER JOIN cust_payment_tx_stream p              ------ Streaming Source: Kinesis data streams
               ON p.customer_id = c.customer_id
WHERE  o.o_orderpriority = '1-URGENT'
GROUP  BY 1,
          2,
          3
ORDER  BY 4 DESC,
          5 DESC
LIMIT  100; 
```

![Analyze](/images/5.Analyze/1.png)

**Alternative** - Using RDS for MySQL Zero-ETL integration with Amazon Redshift.

![Analyze](/images/5.Analyze/2.png)

Congratulations! You completed Zero-ETL approach for a sample analytic campaign. Way to go data explorers!