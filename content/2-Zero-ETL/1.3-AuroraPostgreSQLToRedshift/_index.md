---
title : "Zero-ETL integration between Aurora PostgreSQL and Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 3 
chapter : false
pre : " <b> 2.3 </b> "
---

Announced at [AWS re:Invent 2023](https://youtu.be/PMfn9_nTDbM?t=7418), the feature is now **generally available (GA)** in [multiple regions](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/zero-etl.html#zero-etl.regions) and supports [Aurora PostgreSQL version 16](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL.html#Concepts.Aurora_Fea_Regions_DB-eng.Feature.Zero-ETL-Postgres).

## Benefits

With [Zero-ETL integration between Aurora PostgreSQL and Amazon Redshift](https://aws.amazon.com/rds/aurora/zero-etl/):

- Combine transactional data from Aurora PostgreSQL with the analytical capabilities of Amazon Redshift.
- Eliminate the need to build and maintain custom ETL pipelines between Aurora and Redshift.
- Replicate data from multiple Aurora PostgreSQL clusters into the same or a new Amazon Redshift instance for holistic insights.
- Continuous and near real-time replication of updates from Aurora PostgreSQL to Amazon Redshift.
- The system can be serverless and auto-scale based on data volume, reducing infrastructure management.

## Workflow

- Data is replicated from the source Aurora PostgreSQL database to the target Redshift data warehouse.
- Data becomes available in Amazon Redshift within seconds.
- Enables advanced analytics using Redshift features such as:
  - Data sharing
  - Automatic workload optimization
  - Concurrency scaling
  - Machine learning, and more
- Supports real-time transactional processing in Aurora and simultaneous analytics in Redshift.
- Integration pipelines are monitored and automatically recover from failures.
- You can integrate data from multiple Aurora PostgreSQL clusters into a single Amazon Redshift namespace for centralized insights.

## Pricing

{{% notice info %}}
When using Zero-ETL integration, you **only pay** for the Aurora PostgreSQL and Amazon Redshift resources you use (including data transfer costs). **There are no additional charges** for the Zero-ETL integration itself.
{{% /notice %}}

You are charged for:
- Aurora PostgreSQL resources used to generate and stream change data.
- Amazon Redshift resources used to process and store the data.

See pricing details:
- [Aurora PostgreSQL Pricing](https://aws.amazon.com/rds/aurora/pricing/)
- [Amazon Redshift Pricing](https://aws.amazon.com/redshift/pricing/)