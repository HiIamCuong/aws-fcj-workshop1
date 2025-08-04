---
title : "Query Data Lake using Redshift Spectrum"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 4.1 </b> "
---

In this lab, we show you how to query data in your Amazon S3 data lake with Amazon Redshift without loading or moving data. We will also demonstrate how you can leverage views which union data in Redshift Managed storage with data in S3. You can query structured and semi-structured data from files in Amazon S3 without having to copy or move data into Amazon Redshift tables. For latest guide on the file types that can be queried with Redshift Spectrum, please refer to [supported data formats](https://docs.aws.amazon.com/redshift/latest/dg/c-spectrum-data-files.html#:~:text=Spectrum%20Regions.-,Data%20formats%20for%20Redshift%20Spectrum,-Redshift%20Spectrum%20supports).

## Use-Case Description
**Objective:** Derive data insights to find the effect of customer credit card data breach by finding out the days in January 2023 with most no. of fradulent transactions

**Data Set Description:** Customer credit card transaction data set by year and month

**Data Set S3 Location:**

**us-east-1 (N. Virginia)** region - "s3://redshift-demos/ri2023/ant307/data/cust_payment_tx_history/" [us-east-1_s3_url](https://s3.console.aws.amazon.com/s3/buckets/redshift-demos?region=us-east-1&prefix=ri2023/ant307/data/cust_payment_tx_history/) 

**us-west-2 (Oregon)** region - "s3://redshift-immersionday-labs/ri2023/ant307/data/cust_payment_tx_history/" [us-west-2_s3_url](https://s3.console.aws.amazon.com/s3/buckets/redshift-immersionday-labs?region=us-west-2&prefix=ri2023/ant307/data/cust_payment_tx_history/) 

Below is an overview of the use case steps involved in this lab.

## Instructions
1. Run Glue crawler to populate Glue data catalog

As part of this lab, we will perform following activities:

+ Query historical data residing on S3 by creating an external DB for Redshift Spectrum.

+ Introspect the historical data, perhaps rolling-up the data in novel ways to see trends over time, or other dimensions.

Note the partitioning scheme is Year, Month . Here's a Screenshot: https://s3.console.aws.amazon.com/s3/buckets/redshift-demos?region=us-west-2&prefix=ri2023/ant307/data/cust_payment_tx_history/

![ProducingData](/images/4.FederatedQuery/1.png)

### Create external schema (and DB) for Redshift Spectrum

You can create an external table in Amazon Redshift, AWS Glue, Amazon Athena, or an Apache Hive metastore. If your external table is defined in AWS Glue, Athena, or a Hive metastore, you first create an external schema that references the external database. Then, you can reference the external table in your SELECT statement by prefixing the table name with the schema name, without needing to create the table in Amazon Redshift.

In this lab, you will use AWS Glue Crawler to create external table ant307_external.cust_payment_tx_history stored in parquet format under location s3://us-east-1.redshift-demos/ri2023/ant307/data/cust_payment_tx_history/

+ Navigate to the **Glue Crawler Page**.

us-east-1 region - https://us-east-1.console.aws.amazon.com/glue/home 

us-west-2 region - https://us-west-2.console.aws.amazon.com/glue/home?region=us-west-2 

As part of this lab, we have already created the Glue crawler. Select the crawler - CustPaymentTxHistory and click Run.

![ProducingData](/images/4.FederatedQuery/2.png)

![ProducingData](/images/4.FederatedQuery/3.png)

+ After Crawler run completes, you can see a new table cust_payment_tx_history in Glue Catalog

![ProducingData](/images/4.FederatedQuery/4.png)

2. Create external schema ant307_external in Redshift and select from Glue catalog table - cust_payment_tx_history

Go to Redshift console.
us-east-1 region - https://us-east-1.console.aws.amazon.com/redshiftv2/home?region=us-east-1#/serverless-setup 

us-west-2 region - https://us-west-2.console.aws.amazon.com/redshiftv2/home?region=us-west-2#/serverless-setup 

Click on **Serverless dashboard** menu item to the left side of the console. Click on the name space provisioned earlier. Click **Query data**.

![Validations & Monitoring](/images/2.Zero-ETLIntegration/34.png)

Create an external schema **ant307_external** pointing to your Glue Catalog Database **spectrumdb**.

```
CREATE external SCHEMA ant307_external
FROM data catalog DATABASE 'spectrumdb'
IAM_ROLE default
CREATE external DATABASE if not exists;
```

![ProducingData](/images/4.FederatedQuery/6.png)

### Find the month with most no. of fradulent transactions
You can query the table cust_payment_tx_history, defined in Glue Catalog from Redshift external schema. In January 2023, there is a date which had the most number of fradulent transactions. Can you find that date?

```
SELECT TO_CHAR(TX_DATETIME, 'YYYY-MM-DD'),sum(tx_fraud)
FROM ant307_external.cust_payment_tx_history
WHERE d_year = 2023 and d_month = 01
GROUP BY 1
ORDER BY 2 desc;
```

![ProducingData](/images/4.FederatedQuery/7.png)

3. Create internal schema ant307_das

+ Create a schema ant307_das for tables that will reside on the Redshift Managed Storage.

```
CREATE SCHEMA ant307_das;
```

![ProducingData](/images/4.FederatedQuery/8.png)

4. Run CTAS to create and load Redshift table ant307_das.cust_payment_tx_202301 by selecting from external table

```
CREATE TABLE ant307_das.cust_payment_tx_202301 AS
SELECT *
FROM ant307_external.cust_payment_tx_history
WHERE d_year = 2023 AND d_month = 1;
```

![ProducingData](/images/4.FederatedQuery/9.png)

5. Drop 202301 partitions from external table

Now that we've loaded all January, 2023 data, we can remove the partitions from the Spectrum table so there is no overlap between the Redshift Managed Storage (RMS) table and the Spectrum table.

```
ALTER TABLE ant307_external.cust_payment_tx_history DROP PARTITION(d_year=2023, d_month=1);
```

![ProducingData](/images/4.FederatedQuery/10.png)

6. Create combined view public.ant307_view_cust_payment_tx

```
CREATE VIEW ant307_view_cust_payment_tx AS
  SELECT * FROM ant307_das.cust_payment_tx_202301
  UNION ALL
  SELECT * FROM ant307_external.cust_payment_tx_history
WITH NO SCHEMA BINDING;
```

![ProducingData](/images/4.FederatedQuery/11.png)

### Explain displays the execution plan for a query statement without running the query

+ Note the use of the partition columns in the SELECT and WHERE clauses. Where were those columns in your Spectrum table definition?

+ Note the filters being applied either at the partition or file levels in the Spectrum dataset of the query (versus the Redshift Managed Storage dataset).

+ If you actually run the query (and not just generate the explain plan), does the runtime surprise you? Why or why not?

```
EXPLAIN
SELECT d_year, d_month, COUNT(*)
FROM ant307_view_cust_payment_tx
WHERE d_year = 2023 AND d_month IN (1,2) and terminal_id = 9999
GROUP BY 1,2 
ORDER BY 1,2;
```

![ProducingData](/images/4.FederatedQuery/13.png)