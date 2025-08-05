---
title : "Load the dataset"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 5.2.1 </b> "
---

Typically when running Machine Learning models we need to combine both the live data and historical data, to pick up on trends and detect fraudelent transactions.

We will start in this lab by loading a historical table from an S3 bucket. Compared to the streaming data ingested earlier, the historical data has many more fields than what the streaming data source has.

This will include data such as the customers most recent spend and a risk score to determine the fraud risk. There are also some more variables included such as, weekend transactions or nighttime transactions. These variables we will include are core to picking up those trends as referred to earlier

## Create and load historical data

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

5. Lets create and load data into a transaction history table with the following SQL command:
Data Set S3 Location:

US East (N. Virginia) - us-east-1 region - s3://redshift-demos/ri2023/ant307/data/cust_payment_tx_history/

US West (Oregon) - us-west-2 region - s3://redshift-immersionday-labs/ri2023/ant307/data/cust_payment_tx_history/

```
CREATE TABLE cust_payment_tx_history
(
    TRANSACTION_ID integer,
    TX_DATETIME timestamp,
    CUSTOMER_ID integer,
    TERMINAL_ID integer,
    TX_AMOUNT decimal(9,2),
    TX_TIME_SECONDS integer,
    TX_TIME_DAYS integer,
    TX_FRAUD integer,
    TX_FRAUD_SCENARIO integer,
    TX_DURING_WEEKEND integer,
    TX_DURING_NIGHT integer,
    CUSTOMER_ID_NB_TX_1DAY_WINDOW decimal(9,2),
    CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW decimal(9,2),
    CUSTOMER_ID_NB_TX_7DAY_WINDOW decimal(9,2),
    CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW decimal(9,2),
    CUSTOMER_ID_NB_TX_30DAY_WINDOW decimal(9,2),
    CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW decimal(9,2),
    TERMINAL_ID_NB_TX_1DAY_WINDOW decimal(9,2),
    TERMINAL_ID_RISK_1DAY_WINDOW decimal(9,2),
    TERMINAL_ID_NB_TX_7DAY_WINDOW decimal(9,2),
    TERMINAL_ID_RISK_7DAY_WINDOW decimal(9,2),
    TERMINAL_ID_NB_TX_30DAY_WINDOW decimal(9,2),
    TERMINAL_ID_RISK_30DAY_WINDOW decimal(9,2)
);

COPY cust_payment_tx_history
FROM '<USE S3 LOCATION AS INDICATED ABOVE>'
IAM_ROLE default
PARQUET;
```

![ML](/images/6.ML/1.png)

![ML](/images/6.ML/2.png)

6. Now check how many transactions have been loaded:

```
SELECT 
    count(1) 
FROM 
    cust_payment_tx_history;
```

![ML](/images/6.ML/3.png)

7. Then check the monthly fraud and non fraud transactions trend:

```
SELECT 
    to_char(tx_datetime, 'YYYYMM') AS YearMonth,
    sum(case when tx_fraud=1 then 1 else 0 end) AS fraud_tx,
    sum(case when tx_fraud=0 then 1 else 0 end) AS non_fraud_tx,
    count(*) AS total_tx
FROM cust_payment_tx_history
GROUP BY YearMonth;
```

![ML](/images/6.ML/4.png)

**Congratulations!** You have now setup a table within your Redshift cluster. We can now start building the ML model to make predictions.