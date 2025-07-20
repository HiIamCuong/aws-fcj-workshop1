---
title : "Zero-ETL Integration Aurora MySQL to Redshift"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

Announced at AWS [re:Invent 2023](https://youtu.be/PMfn9_nTDbM?t=7418) and now generally available ([GA](https://aws.amazon.com/about-aws/whats-new/2024/09/amazon-rds-mysql-zero-etl-integration-redshift-generally-available/)) in many regions for RDS for MySQL version 8.0.

## Benefits

With the **[Zero-ETL Integration](1-Zero-ETL/)** feature between **RDS for MySQL** and **Amazon Redshift**, you can combine transactional data from **RDS for MySQL** with the powerful analytics capabilities of **Amazon Redshift**. This feature minimizes the work of building and managing custom ETL pipelines between **RDS for MySQL** and **Amazon Redshift**. Data engineers can now replicate data from multiple **RDS for MySQL** database clusters into the same or a new **Amazon Redshift** cluster to gain overall insights across multiple applications or partitions. Updates in **RDS for MySQL** are automatically and continuously pushed to **Amazon Redshift**, ensuring that data engineers always have the latest information in near real-time. Additionally, the entire system can be serverless (**serverless**) and is capable of auto-scaling up or down based on data volume, so you don’t have to manage infrastructure.

## Progress

This integration replicates data from the source database to the destination data warehouse. Data will be available in **Amazon Redshift** in just a few seconds, enabling users to leverage Amazon Redshift's analytics features and capabilities like data sharing, automated workload optimization, concurrent scaling, machine learning, and more. You can process real-time transactions on data in **RDS for MySQL** while simultaneously using **Amazon Redshift** for analytics workloads like reporting and dashboards.

The integration will monitor the status of the data pipeline and automatically resolve issues where possible. You can create integrations from multiple **RDS for MySQL** clusters into a single **Amazon Redshift namespace**, enabling analytics across multiple applications.

## Cost

{{% notice notice about cost %}}
When you create a **Zero-ETL** integration between **RDS for MySQL** and **Amazon Redshift**, you will still pay the current pricing for **RDS MySQL** and **Amazon Redshift** (including data transfer costs). This integration feature does not incur additional charges.
{{% /notice %}}

You will be charged for the existing **Amazon RDS for MySQL** and **Amazon Redshift** resources used to create and process data changes during the **Zero-ETL** integration. For more information, see the pricing for [Amazon RDS for MySQL](https://aws.amazon.com/rds/mysql/pricing/) and [Amazon Redshift](https://aws.amazon.com/redshift/pricing/).
