+++
title = "Clean up resources"
date = 2022
weight = 6
chapter = false
pre = "<b>6. </b>"
+++

{{% notice info %}}  
You **MUST** clean up in your **own AWS account** to avoid any runaway costs.
{{% /notice %}}

We will take the following steps to delete the resources we created in this exercise.

## 1. Delete Zero-ETL integration
Delete the zero-etl integrations created as part of this workshop named **zero-etl-***, follow the steps below for deletion:

+ On the **Amazon RDS console**, choose **Zero-ETL integrations** in the navigation pane.
+ Select the **zero-ETL integration** that you want to delete and choose **Delete**. Do this for **ALL** integrations created.
+ To confirm the deletion, type in box `confirm` then choose **Delete**.

![Clean](/images/7.clean/1.png)

![Clean](/images/7.clean/2.png)

![Clean](/images/7.clean/3.png)

## 2. Delete Stack 

{{% notice info %}}  
You need check and **empty** **S3 bucket** have name **zero-etl-*** first because CloudFormation cannot **empty** bucket. That will make error when delete **stack**. If you don't empty **S3 bucket** first and delete **stack** this will happend
{{% /notice %}}

![Clean](/images/7.clean/8.png)

1. Empty S3 bucket

+ Go to the **S3 console** choose **Bucket**
+ Choose bucket with prefix name **zero-etl-*** 
+ Choose **Empty**

![Clean](/images/7.clean/6.png)

+ type `permanently delete`
+ Choose **Empty**

![Clean](/images/7.clean/5.png)

![Clean](/images/7.clean/7.png)

2. Delete Stack

+ Go to **CloudFormation Console**
+ Choose **Stacks**
+ Choose **zero-etl-lab** (dont have NESTED text on name)

![Clean](/images/7.clean/4.png)

![Clean](/images/7.clean/11.png)

## 3. Delete DB snapshots

+ Go in **Aurora and RDS console**
+ Choose snapshots in left pane
+ Check and **delete** all snapshots **manual snapshots** and **system snapshots** related to the lab (have prefix name **zero-etl**)

![Clean](/images/7.clean/12.png)

{{% notice info %}}  
**Congratulations** you have completed the **clean up** and completed the workshop. Thank you for taking the time to do this.
{{% /notice %}}