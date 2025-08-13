---
title : "Session Management"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Connect and analyze all your data with zero-ETL approaches

### Overall

In this lab, we will use the zero-ETL feature of [Amazon Redshift](https://aws.amazon.com/redshift/) to consolidate data from multiple source database clusters of [Amazon Aurora](https://aws.amazon.com/rds/aurora/) into Redshift data warehouses. Data in [Amazon Redshift](https://aws.amazon.com/redshift/) can be updated from [Amazon Aurora](https://aws.amazon.com/rds/aurora/) clusters in near real-time.

After that we will use some feature of Redshift to help us analyse all our data and make decision for your business with: 
- [**Redshift Streaming Ingestion**](3-RedshiftStreamingIngestion/) – Stream data ingestion into Redshift  
- [**Federated Query**](4-FederatedQueries) – Query data from multiple sources  
- [**Amazon Redshift ML**](5-Analytics&ML) – Integrate machine learning for predictions and advanced analytics  

![Architecture](/images/architecture.png)

### Content
 1. [Prerequisite](1-Prerequisite/)
 2. [Zero-ETL](2-Zero-ETL/)
 3. [Redshift Streaming Ingestion](3-RedshiftStreamingIngestion/)
 4. [Federated Queries](4-FederatedQueries)
 5. [Analytics & Machine Learning](5-Analytics&ML)
 6. [Clean up resources](6-cleanup/)
