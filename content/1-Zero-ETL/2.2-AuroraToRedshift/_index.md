---
title : "Zero-ETL Integration Aurora Amazon Aurora to Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1.2 </b> "
---

Announced at [AWS re:Invent 2022](https://youtu.be/Xus8C2s5K9A?t=2212), the feature is now generally available ([GA](https://aws.amazon.com/about-aws/whats-new/2023/11/aws-general-availability-amazon-aurora-mysql-zero-etl-integration-redshift/)) in [multiple regions](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html) for [Amazon Aurora MySQL-Compatible Edition 3](https://aws.amazon.com/rds/aurora/mysql-features/) (version 3.05.2 or later, compatible with MySQL 8.0.32 or higher — be sure to verify the [latest supported versions](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html)).

## Benefits

By using [Aurora Zero-ETL Integration with Amazon Redshift](1-Zero-ETL/), you can combine transactional data from Aurora with Amazon Redshift’s analytical capabilities. This reduces the effort of building and managing custom ETL pipelines between Aurora and Redshift.

Data engineers can replicate data from multiple Aurora database clusters into the same or a new Amazon Redshift instance to gain a holistic view across multiple applications or partitions. Updates in Aurora are automatically and continuously replicated to Redshift, providing near real-time insights.

The entire system can run serverless and automatically scale in or out based on workload, eliminating infrastructure management.

## Workflow

With Aurora Zero-ETL integration with Amazon Redshift, data is replicated from the source database to the target data warehouse. The data becomes available in Redshift within seconds, enabling the use of features such as:

- Data sharing
- Automatic workload optimization
- Concurrency scaling
- Machine learning integration

You can continue to perform real-time transactions on Aurora while using Redshift for analytics, dashboards, and reporting.

The integration monitors the pipeline health and attempts recovery from failures when possible. You can integrate multiple Aurora DB clusters into a single Amazon Redshift namespace for cross-application insights.

## Pricing

> When you create an Aurora Zero-ETL integration with Amazon Redshift, you continue to pay for Aurora and Amazon Redshift usage at their current rates (including data transfer costs). The integration feature itself incurs **no additional charge**.

You will pay for the following resources:

- Additional I/O and storage due to enhanced binary logging
- Snapshot export for initial data load into Redshift
- Extra Amazon Redshift storage for replicated data
- Data transfer between Availability Zones (AZs)

**You are not charged** for continuous change data replication.

For more information, see:

- [Amazon Aurora Pricing](https://aws.amazon.com/rds/aurora/pricing/)
- [Amazon RDS for MySQL Pricing](https://aws.amazon.com/rds/mysql/pricing/)
- [Amazon Redshift Pricing](https://aws.amazon.com/redshift/pricing/)
