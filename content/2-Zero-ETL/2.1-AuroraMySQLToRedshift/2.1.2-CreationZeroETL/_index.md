---
title : "Zero-ETL Integration Aurora MySQL to Amazon Redshift"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.1.2 </b> "
---

## Tạo tích hợp Zero-ETL
1. To create the zero-ETL integration
+ Access Amazon RDS console
+ Choose Zero-ETL integrations in the navigation pane
+ Click on Create zero-ETL integration as shown below.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/10.png)

2. Under **Step 1** Getting started, for **Integration identifier**
+ Enter an integration name of your choice, for example `zero-etl-aurora-mysql-redshift`. 
+ Click Next.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/11.png)

3. Under Step2 Select source 
+ Click on Browse RDS databases
+ Choose the source aurora mysql cluster named **zero-etl-lab-sourceamscluster-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/12.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/13.png)

Since source pre-requisites were not already met, we will get an error message along with a **Fix it for me (requires reboot)** checkbox. You will also see an option to apply **DATA FILTERING**. Take following actions:

+ Check **Fix it for me (requires reboot)** check box.
+ Check the **Customize data filtering options** check box under **Data filtering options**.
+ For **Include** filter type, use `*.*` as the filter expression. Ignore if already populated.
+ Click on the **Add Filter** button to add an exclusion criteria as: **Exclude** values like `filter_missingpk.*`. This will prevent any tables in schema/database named **filter_missingpk** flow through the zero-etl integration.
+ When complete, click **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/14.png)

Fix it for me starts working! You will be presented with the changes on the source cluster that will be applied. It would create a new *DbClusterParameterGroup* with name **zero-etl-params-*** setting the 6 parameters automatically updated with required values. This will also require a reboot of your cluster, 
+ So you would have to type **confirm** and click **Reboot and continue**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/15.png)

+ The status bar shows your source cluster being modified. You can click on **View details** button to view the changes.

+ When complete, the status bar will turn green and the **View details** button will show all required changes as *Complete*.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/18.png)

4. Under Step 3 Select target 
+ Click on **Browse Redshift data warehouses**

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/19.png)

+ Choose the Redshift Serverless destination namespace **zero-etl-destination-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/20.png)

Since target pre-requisites were not already done, we will get an error message along with a **Fix it for me** checkbox underneath.
+ Check *Fix it for me*
+ click Next.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/21.png)

Now you will be presented with **Review changes** screen. It would present you with the **Resource policy** and **case sensitivity parameter changes**. 

+ You would click on Continue and proceed.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/22.png)

+ The status bar will show target datawarehouse being modified. The target changes would complete quickly and status bar turns green. You can click on **View details** button to view the completed changes. The changes happen in background, doesn't prevent you from continuing further.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/23.png)

5. Under optional Step 4 Add tags and encryption
+ Take no action and click **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/24.png)

6. Under Step 5 Review and create, do the following
+ Review to ensure all pre-requisites were handled by the **Fix it for me** button. You can check the progress of the changes using **View Details** button in the progress bar at the top of console. You may have to wait for a couple of minutes as source changes **requires reboot** which takes a couple of minutes.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/25.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/26.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/27.png)

You can click on the integration to view the details and monitor its progress. It takes a few minutes to change the status from Creating to Active. The time varies depending on size of the dataset already available in the source.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/28.png)

During the process of zero-ETL integration creation the Redshift serverless workgroup and namespace status go from Available to Modifying to Available. That's normal and expected.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/29.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/30.png)

{{% notice info %}}  
It will take about **15-30 minutes** to create zero-ETL integration. Let's discuss the zero-ETL vision and any other questions so far. You may also consider working on other labs like **Streaming Ingestion** or **Federated Query** in the meantime. **Note** you cannot create multiple zero-etl integrations in parallel to the same destination. If an existing integration is **Creating**, next one has to wait for it to finish.
{{% /notice %}}

