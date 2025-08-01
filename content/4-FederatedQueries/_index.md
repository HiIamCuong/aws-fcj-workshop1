---
title : "Federated Queries in Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 4 
chapter : false
pre : " <b> 4. </b> "
---

By using **federated queries** in Amazon Redshift, you can query and analyze data across operational databases, data warehouses, and data lakes. With the **Federated Query** feature, you can integrate queries from Amazon Redshift on live data in external databases with queries on your **Amazon Redshift** and **Amazon S3** environments.

Federated queries work with external databases such as:

- **Amazon RDS for PostgreSQL**
- **Amazon Aurora PostgreSQL-Compatible Edition**
- **Amazon RDS for MySQL**
- **Amazon Aurora MySQL-Compatible Edition**

You can use federated queries to combine live data as part of business intelligence (BI) and reporting applications. For example, to simplify loading data into Amazon Redshift, you can use federated queries to:

- Query operational databases directly
- Apply quick transformations
- Load data into destination tables without complex ETL (Extract, Transform, Load) processes

To reduce data movement over the network and improve performance, Amazon Redshift pushes part of the computation for federated queries directly to remote operational databases. It also uses its **parallel processing** capabilities to support execution of these queries when needed.

## How Federated Queries Work

When running a federated query, Amazon Redshift performs the following steps:

1. The **leader node** creates a client connection to the RDS or Aurora DB instance to retrieve **table metadata**.
2. A **compute node** issues subqueries with pushed-down predicates and retrieves result rows.
3. Amazon Redshift **distributes the result rows** among the compute nodes for further processing.

This process allows for efficient and scalable querying across multiple data sources.

