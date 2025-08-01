---
title : "Preparation "
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

{{% notice info %}}  
First, you need to log in to your **AWS Management Console** account with **Administrator** privileges and select the **N. Virginia (us-east-1)** or **Oregon (us-west-2)** region.  
{{% /notice %}}

### 1. Create the Stack

Click [here](https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=zero-etl-lab&templateURL=https://redshift-demos.s3.amazonaws.com/zetl/approaches/zeroetl.yaml) to go to the **Stack Configuration** page in **AWS CloudFormation**.

(**Stacks** in **AWS CloudFormation** are used to store the configuration of AWS services and allow you to create, launch, and delete multiple AWS services in bulk.)

### 2. After clicking the link, you’ll land on Step 1  
+ Click **Next**  

![Create Stack](/images/2.prerequisite/1.png)

### 3. In Step 2  
+ Click **Next**  

![Create Stack](/images/2.prerequisite/2.png)  
![Create Stack](/images/2.prerequisite/3.png)

### 4. In Step 3  
+ Check **I acknowledge that AWS CloudFormation might create IAM resources with custom name**  
+ Check **I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND**  
+ Click **Next**  

![Create Stack](/images/2.prerequisite/4.png)  
![Create Stack](/images/2.prerequisite/5.png)  
![Create Stack](/images/2.prerequisite/6.png)

### 5. In Step 4  
+ Review the information  
+ Click **Next**  

![Create Stack](/images/2.prerequisite/7.png)  
![Create Stack](/images/2.prerequisite/8.png)  
![Create Stack](/images/2.prerequisite/9.png)  
![Create Stack](/images/2.prerequisite/10.png)  
![Create Stack](/images/2.prerequisite/11.png)

### 6. Wait for the Stack to finish creating all resources  

![Create Stack](/images/2.prerequisite/12.png)

### 7. Completed  

![Create Stack](/images/2.prerequisite/13.png)

{{% notice info %}}  
At this point, the preparation is complete. You can proceed to **Part 3**. The section below provides additional explanations about the resources created by this stack.  
{{% /notice %}}

---

- A **VPC** with multiple **subnets**, **route tables**, and other networking resources such as **Security Groups** for **EC2**, **RDS**, and **Aurora** services.

![Create Stack](/images/2.prerequisite/14.png)

- An **Amazon S3 bucket** named `zero-etl-approaches-file-store-*` to store Machine Learning directories.

![Create Stack](/images/2.prerequisite/15.png)

- **IAM Roles** with the required privileges to execute queries on **Amazon RDS** and **Amazon Redshift**.

![Create Stack](/images/2.prerequisite/35.png)

- An **EC2 instance** for connecting to and executing queries on **Aurora MySQL** and **Aurora PostgreSQL**.

![Create Stack](/images/2.prerequisite/16.png)  
![Create Stack](/images/2.prerequisite/17.png)  
![Create Stack](/images/2.prerequisite/18.png)  
![Create Stack](/images/2.prerequisite/19.png)  
![Create Stack](/images/2.prerequisite/20.png)

- A **Glue Crawler** named `CustPaymentTxHistory` to extract data from **Amazon S3** and store it in a **Glue Database** named `spectrumdb`.

![Create Stack](/images/2.prerequisite/21.png)  
![Create Stack](/images/2.prerequisite/22.png)

- A **Kinesis Data Stream** named `cust-payment-txn-stream`.

![Create Stack](/images/2.prerequisite/23.png)

- A **Lambda Function** named `GenStreamData-zero-etl-lab` that sends data to the Kinesis Data Stream.

![Create Stack](/images/2.prerequisite/24.png)

- An **AWS EventBridge Rule** named `zero-etl-lab-EventRule-*` to trigger the Lambda function (above) every 1 minute when activated.

![Create Stack](/images/2.prerequisite/25.png)

- **(Nested Stack)** **Amazon Aurora MySQL 3.05.0 (or higher)** *(compatible with MySQL 8.0.32 or higher)*, with `DbInstanceClass: Serverless V2`, and a new **DbClusterParameterGroup**. Each resource name is prefixed with `zero-etl-lab-sourceamscluster-*`.

![Create Stack](/images/2.prerequisite/27.png)  
![Create Stack](/images/2.prerequisite/26.png)

- **(Nested Stack)** **Amazon RDS MySQL 8.0.36 (or higher)**, `DbInstance` named with the prefix `zero-etl-lab-sourcerdsmscluster-*`.

![Create Stack](/images/2.prerequisite/28.png)  
![Create Stack](/images/2.prerequisite/30.png)

- **(Nested Stack)** **Amazon Aurora PostgreSQL 13.9**, `DbInstanceClass: Serverless V1`, with **cluster/instance** name `aurorapg`.

![Create Stack](/images/2.prerequisite/34.png)  
![Create Stack](/images/2.prerequisite/33.png)
