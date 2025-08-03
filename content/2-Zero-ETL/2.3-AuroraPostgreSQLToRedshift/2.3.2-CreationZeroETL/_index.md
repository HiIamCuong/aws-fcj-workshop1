---
title : "Creation Zero-ETL"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 2.3.2 </b> "
---

## Create Zero-ETL Integration
1. To create the zero-ETL integration
+ Access **Amazon RDS** console
+ Choose **Zero-ETL integrations** in the navigation pane
+ Click on **Create zero-ETL integration** as shown below.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/10.png)

2. Under **Step 1** Getting started, for **Integration identifier**
+ Enter an integration name of your choice, for example `zero-etl-aurora-postgres-redshift`. 
+ Click **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/117.png)

3. Under Step2 Select source 
+ Click on Browse RDS databases
+ Choose the source aurora mysql cluster named **aurorapg-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/12.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/118.png)

Since source pre-requisites were not already met, we will get an error message along with a **Fix it for me (requires reboot)** checkbox. You will also see an option to apply **DATA FILTERING**. Take following actions:

+ Check **Fix it for me (requires reboot)** check box.
+ Check the **Customize data filtering options** check box under **Data filtering options**.
+ For **Include** filter type, use `postgres.public.*` as the filter expression. Ignore if already populated.
+ Click on the **Add Filter** button to add an exclusion criteria as: **Exclude** values like `postgres.public.customer_info_missing_pk`. This will prevent any tables in schema/database named **customer_info_missing_pk** flow through the zero-etl integration.
+ When complete, click **Next**.

{{% notice info %}}
**Data Filtering** using *** wildcard** is applicable for ALL or NOTHING scenarios for **schemas** or **tables**. If you want to string match **starts with** or **contains** kind of values, use regular expressions in the **include** and **exclude** clauses using [maxwell filter syntax](https://maxwells-daemon.io/filtering/) instead. Read more in [data filtering for zero etl](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/zero-etl.filtering.html#zero-etl.filtering-format) documentation.
{{% /notice %}}

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/119.png)

Fix it for me starts working! You will be presented with the changes on the source cluster that will be applied. It would create a new *ParameterGroup* with name **zero-etl-params-*** setting the 2 parameters automatically updated with required values. This will also require a reboot of your cluster, 
+ So you would have to type **confirm** in box click **Reboot and continue**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/120.png)

+ The status bar shows your source cluster being modified. You can click on **View details** button to view the changes.

+ When complete, the status bar will turn green and the **View details** button will show all required changes as *Complete*.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/121.png)

4. Under Step 3 Select target 
+ Click on **Browse Redshift data warehouses**

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/19.png)

+ Choose the Redshift Serverless destination namespace **zero-etl-destination-***.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/20.png)

Since target pre-requisites were not already done, we will get an error message along with a **Fix it for me** checkbox underneath.
+ Check *Fix it for me*
+ click Next.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/122.png)

Now you will be presented with **Review changes** screen. It would present you with the **Resource policy** and **case sensitivity parameter changes**. 

+ You would click on **Continue** and proceed.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/123.png)

+ The status bar will show target datawarehouse being modified. The target changes would complete quickly and status bar turns green. You can click on **View details** button to view the completed changes. The changes happen in background, doesn't prevent you from continuing further.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/124.png)

5. Under optional Step 4 Add tags and encryption
+ Take no action and click **Next**.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/24.png)

6. Under Step 5 Review and create, do the following
+ Review to ensure all pre-requisites were handled by the **Fix it for me** button. You can check the progress of the changes using **View Details** button in the progress bar at the top of console. You may have to wait for a couple of minutes as source changes **requires reboot** which takes a couple of minutes.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/125.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/126.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/127.png)

You can click on the integration to view the details and monitor its progress. It takes a few minutes to change the status from **Creating** to **Active**. The time varies depending on size of the dataset already available in the source.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/128.png)

During the process of zero-ETL integration creation the Redshift serverless workgroup and namespace status go from Available to Modifying to Available. That's normal and expected.

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/29.png)

![Create Zero-ETL Integration](/images/2.Zero-ETLIntegration/30.png)

{{% notice info %}}  
It will take about **15-30 minutes** to create zero-ETL integration. Let's discuss the zero-ETL vision and any other questions so far. You may also consider working on other labs like **Streaming Ingestion** or **Federated Query** in the meantime. **Note** you cannot create multiple zero-etl integrations in parallel to the same destination. If an existing integration is **Creating**, next one has to wait for it to finish.
{{% /notice %}}