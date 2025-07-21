---
title : "Zero-ETL Integration Aurora MySQL to Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1.1 </b> "
---

Announced at [AWS re:Invent 2023](https://youtu.be/PMfn9_nTDbM?t=7418) and now generally available ([GA](https://aws.amazon.com/about-aws/whats-new/2024/09/amazon-rds-mysql-zero-etl-integration-redshift-generally-available/)) in many regions for **RDS for MySQL version 8.0**.

## Benefits

With the **[Zero-ETL Integration](1-Zero-ETL/)** feature between **RDS for MySQL** and **Amazon Redshift**, you can combine transactional data from **RDS for MySQL** with the powerful analytics capabilities of **Amazon Redshift**. This feature minimizes the work of building and managing custom ETL pipelines between the two systems.

Data engineers can replicate data from multiple **RDS for MySQL** database clusters into the same or a new **Amazon Redshift** cluster to gain a consolidated view across multiple applications or partitions. Updates in **RDS for MySQL** are automatically and continuously pushed to **Amazon Redshift**, ensuring near real-time availability of up-to-date information.

The entire system can run in **serverless** mode and is capable of **auto-scaling** based on data volume, eliminating infrastructure management overhead.

## Progress

This integration replicates data from the source database to the destination data warehouse. Data becomes available in **Amazon Redshift** within seconds, allowing users to take advantage of features such as:

- Data sharing  
- Automated workload optimization  
- Concurrency scaling  
- Machine learning  
- And more

You can perform real-time transactional processing on **RDS for MySQL** data while simultaneously using **Amazon Redshift** for analytics workloads like reporting and dashboards.

The integration monitors the health of the data pipeline and attempts to automatically resolve issues when possible. You can integrate multiple **RDS for MySQL** clusters into a single **Amazon Redshift namespace**, enabling cross-application analytics.

## Cost

{{% notice info %}}
When you create a **Zero-ETL** integration between **RDS for MySQL** and **Amazon Redshift**, you will continue to pay for **RDS for MySQL** and **Amazon Redshift** usage at their current rates (including data transfer costs). This integration feature does **not incur additional charges**.
{{% /notice %}}

You will be billed for the existing **Amazon RDS for MySQL** and **Amazon Redshift** resources used to create and process data changes during the **Zero-ETL** integration.

For more details, refer to:

- [Amazon RDS for MySQL Pricing](https://aws.amazon.com/rds/mysql/pricing/)
- [Amazon Redshift Pricing](https://aws.amazon.com/redshift/pricing/)
