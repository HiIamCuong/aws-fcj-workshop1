---
title : "Producing the Data"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 3.1 </b> "
---

In this lab we will send data to the Kinesis Data Stream using a combination of EventBridge and Lambda

We have provisoned three components for you already as part of this lab:

+ A Kinesis Data Stream - **cust-payment-txn-stream**

+ An EventBridge rule - **zero-etl-lab-EventRule**

+ A Lambda function - **GenStreamData-zero-etl-lab**

The Lambda generates mock customer financial transaction data to send to the Kinesis Data Stream and is triggered by EventBridge to generate random data every minute. Lets setup this flow and check that the stream receives the data

1. Navigate to [EventBridge](https://console.aws.amazon.com/events/home)  in the AWS console

2. Go to **Rules** 

3. Select and click on the rule that starts with **zero-etl-lab-EventRule**

4. Click the **Enable** button

![ProducingData](/images/3.RedshiftStreamingIngestion/1.png)

![ProducingData](/images/3.RedshiftStreamingIngestion/2.png)

![ProducingData](/images/3.RedshiftStreamingIngestion/3.png)

EventBridge will now trigger the Lambda function to generate data every 1 minute

5. Navigate now to [Kinesis](https://console.aws.amazon.com/kinesis/home) 

6. Select **Data streams** on the left hand side menu

7. Select and click on the stream named **cust-payment-txn-stream**

8. Navigate to the monitoring tab of the stream

{{% notice info %}}
Note: If you do not see the metrics updating or the dashboard is empty, it can take up to 15 mins to update. Please carry on with the labs and come back to this part later.
{{% /notice %}}

![ProducingData](/images/3.RedshiftStreamingIngestion/6.png)

9. Once you see data flowing into the stream, please navigate back to [EventBridge](https://console.aws.amazon.com/events/home)  and disable the rule that was enabled earlier

![ProducingData](/images/3.RedshiftStreamingIngestion/7.png)

![ProducingData](/images/3.RedshiftStreamingIngestion/8.png)

Congratulations, you have now setup a working Kinesis Data Stream receiving data from Lambda triggered by EventBridge. Lets move onto creating a view of this data in Redshift.