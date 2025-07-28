---
title : "Redshift Machine Learning"
date :  "`r Sys.Date()`" 
weight : 5 
chapter : false
pre : " <b> 5. </b> "
---

# Amazon Redshift ML

Amazon Redshift ML enables data analysts and database developers to easily create, train, and apply machine learning models using familiar SQL commands within Amazon Redshift data warehouses. With Redshift ML, you can leverage Amazon SageMaker – a fully managed machine learning service – without needing to learn new tools or languages. You simply use SQL commands to create and train machine learning models from Amazon SageMaker using data stored in Redshift, and then apply these models to make predictions.

## Example Use Case
For example, you can use customer retention data within Redshift to train a model that detects at-risk customers, and then apply this model in dashboards for the marketing team to create tailored retention offers. Redshift ML delivers models as SQL functions within the Redshift data warehouse, making them easy to use directly in queries and reports.

### Key Features

#### No Machine Learning Experience Needed
Since Redshift ML allows you to use standard SQL, you can easily develop new use cases for your analytical data. Redshift ML provides simple, optimized, and secure integration between Redshift and Amazon SageMaker, while also supporting inference directly within the Redshift cluster. This eliminates the need to manage a separate inference endpoint, making it easy to use predictions from machine learning models in queries and applications. The training data is also fully encrypted for security.

#### Using Machine Learning on Redshift Data with Standard SQL
To get started, you use the SQL command `CREATE MODEL` in Redshift and specify your training data as a table or `SELECT` query. Redshift ML will compile and import the trained model into the Redshift data warehouse and automatically generate an SQL function for inference that can be used immediately in queries. Redshift ML handles all the necessary steps for model training and deployment.

#### Predictive Analytics with Amazon Redshift
With Redshift ML, you can directly integrate predictions like fraud detection, risk assessment, and customer churn forecasting into your queries and reports. You can run SQL functions like "churn prediction" on new customer data in your data warehouse periodically to predict which customers are at risk of leaving, providing this information to sales and marketing teams for preventative actions – such as sending personalized retention offers.

#### Bring Your Own Model (BYOM)
Redshift ML supports the use of BYOM (Bring Your Own Model) for local or remote inference. You can use a model trained outside of Redshift via Amazon SageMaker to perform inference within Redshift. You can import models trained by SageMaker Autopilot or custom SageMaker models to perform local inference. Alternatively, you can invoke custom machine learning models remotely from SageMaker endpoints. You can use any machine learning model in SageMaker that accepts and returns data in text or CSV format to perform remote inference.

