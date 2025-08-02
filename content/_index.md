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
- [**Redshift Streaming Ingestion**](https://catalog.us-east-1.prod.workshops.aws/workshops/428641a0-1414-4fb7-8de6-a38c053ee19e/en-US/02streaming) – Stream data ingestion into Redshift  
- [**Federated Query**](https://catalog.us-east-1.prod.workshops.aws/workshops/428641a0-1414-4fb7-8de6-a38c053ee19e/en-US/03federated) – Query data from multiple sources  
- [**Amazon Redshift ML**](https://catalog.us-east-1.prod.workshops.aws/workshops/428641a0-1414-4fb7-8de6-a38c053ee19e/en-US/04redshiftml/02rml) – Integrate machine learning for predictions and advanced analytics  


### Content
 1. [Prerequisite](1-Prerequisite/)
 2. [Zero-ETL](2-Zero-ETL/)
 3. [Redshift Streaming Ingestion](3-RedshiftStreamingIngestion/)
 4. [Federated Queries](4-FederatedQueries)
 5. [Analytics & Machine Learning](5-Analytics&ML)
 6. [Clean up resources](6-cleanup/)
