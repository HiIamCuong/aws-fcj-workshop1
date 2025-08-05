---
title : "Building an ML Model"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 5.2.2 </b> "
---

**Amazon Redshift ML** enables you to train models with one single **SQL CREATE MODEL** command. The **CREATE MODEL** method creates a model that Amazon Redshift uses to generate model-based predictions with familiar SQL constructs.

**Amazon Redshift ML** supports data analysts and data scientists in using machine learning. It also makes it possible for machine learning experts to use their knowledge to guide the **CREATE MODEL** statement to use only the aspects that they specify. By doing so, you can speed up the time that **CREATE MODEL** needs to find the best candidate, improve the accuracy of the model, or both.

**CREATE MODEL** uses **Amazon SageMaker Autopilot** behind the scenes. This service performs data preparation, feature engineering, model selection and training automatically for you, without having to create custom pipelines in SageMaker itself.

We will be using CREATE MODEL in this lab in order to train and test a model on our historical card transaction table. This has 6 months worth of data that we can use for the training

The model takes the following fields as input:

```
TX_DURING_WEEKEND ,
TX_AMOUNT,
TX_DURING_NIGHT ,
CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
TERMINAL_ID_NB_TX_1DAY_WINDOW ,
TERMINAL_ID_RISK_1DAY_WINDOW ,
TERMINAL_ID_NB_TX_7DAY_WINDOW ,
TERMINAL_ID_RISK_7DAY_WINDOW ,
TERMINAL_ID_NB_TX_30DAY_WINDOW ,
TERMINAL_ID_RISK_30DAY_WINDOW
```

We get **tx_fraud** as output.

We split this data into training and test datasets. Transactions from **2022-04-01** to **2022-07-31** are for the training set. Transactions from **2022-08-01** to **2022-09-30** are used for the test set.

Let’s create the ML model using the familiar [SQL CREATE MODEL](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_MODEL.html) statement . We use a basic form of the Redshift ML command. The following method uses [Amazon SageMaker Autopilot](https://aws.amazon.com/sagemaker/autopilot/) , which performs data preparation, feature engineering, model selection, and training automatically for you.

Execute the following code (Provide the name of your S3 bucket starting with prefix **zero-etl-approaches-file-store-***).

```
CREATE MODEL cust_cc_txn_fd
FROM (
    SELECT 
        TX_AMOUNT ,
        TX_FRAUD ,
        TX_DURING_WEEKEND ,
        TX_DURING_NIGHT ,
        CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
        CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
        CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
        TERMINAL_ID_NB_TX_1DAY_WINDOW ,
        TERMINAL_ID_RISK_1DAY_WINDOW ,
        TERMINAL_ID_NB_TX_7DAY_WINDOW ,
        TERMINAL_ID_RISK_7DAY_WINDOW ,
        TERMINAL_ID_NB_TX_30DAY_WINDOW ,
        TERMINAL_ID_RISK_30DAY_WINDOW
    FROM cust_payment_tx_history
    WHERE cast(tx_datetime as date) BETWEEN '2022-06-01' AND '2022-09-30'
) 
TARGET tx_fraud
FUNCTION fn_customer_cc_fd
IAM_ROLE default
AUTO OFF
MODEL_TYPE XGBOOST
OBJECTIVE 'binary:logistic'
PREPROCESSORS 'none'
HYPERPARAMETERS DEFAULT EXCEPT (NUM_ROUND '100')
SETTINGS (
    S3_BUCKET '<replace this with your s3 bucket name: zero-etl-approaches-file-store-*>',
    S3_GARBAGE_COLLECT off,
    MAX_RUNTIME 1200
);
```

![ML](/images/6.ML/5.png)

**The ML model** is called **Cust_cc_txn_fd**, and the prediction function is **fn_customer_cc_fd**. The **FROM** clause shows the input columns from the historical table **public.cust_payment_tx_history**. The target parameter is set to **tx_fraud**, which is the target variable that we’re trying to predict. **IAM_Role** is set to default because the cluster is configured with this role; if not, you have to provide your **Amazon Redshift cluster IAM role ARN**.

**MAX_RUNTIME** specifies the maximum amount of time to train. Training jobs often complete sooner depending on dataset size. This specifies the maximum amount of time the training should take. The default is **5,400 (90 minutes)**.

**Redshift ML** deploys the best model that is identified in this time frame. Depending on the complexity of the model and the amount of data, it can take some time for the model to be available. For this workshop we use **1200** to complete the process in **no more than 20 minutes**.

The **CREATE MODEL** command is run asynchronously, which means it runs in the background. You can use the **SHOW MODEL** command to see the status of the model.

During training **"Model State"** is set tot **"TRAINING"**, when **"Model State"** shows as **"READY"**, it means the model is trained and deployed.

```
SHOW MODEL cust_cc_txn_fd;
```

![ML](/images/6.ML/6.png)

![ML](/images/6.ML/7.png)

From the output, I see that the model has been correctly recognized as **BinaryClassification** and have **train:error** very low 

Let’s test this model with the test dataset.

Run the following command, which retrieves sample predictions:

```
SELECT
    tx_fraud ,
    fn_customer_cc_fd(
        TX_AMOUNT ,
        TX_DURING_WEEKEND ,
        TX_DURING_NIGHT ,
        CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
        CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
        CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
        TERMINAL_ID_NB_TX_1DAY_WINDOW ,
        TERMINAL_ID_RISK_1DAY_WINDOW ,
        TERMINAL_ID_NB_TX_7DAY_WINDOW ,
        TERMINAL_ID_RISK_7DAY_WINDOW ,
        TERMINAL_ID_NB_TX_30DAY_WINDOW ,
        TERMINAL_ID_RISK_30DAY_WINDOW
    )
FROM cust_payment_tx_history
WHERE cast(tx_datetime as date) >= '2022-10-01'
LIMIT 10;
```

![ML](/images/6.ML/8.png)

We see that some values are matching and some are not. Let’s compare predictions to the ground truth:

```
SELECT
    tx_fraud ,
    fn_customer_cc_fd(
        TX_AMOUNT ,
        TX_DURING_WEEKEND ,
        TX_DURING_NIGHT ,
        CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
        CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
        CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
        CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
        TERMINAL_ID_NB_TX_1DAY_WINDOW ,
        TERMINAL_ID_RISK_1DAY_WINDOW ,
        TERMINAL_ID_NB_TX_7DAY_WINDOW ,
        TERMINAL_ID_RISK_7DAY_WINDOW ,
        TERMINAL_ID_NB_TX_30DAY_WINDOW ,
        TERMINAL_ID_RISK_30DAY_WINDOW
    ) AS prediction, 
    count(*) AS values
FROM public.cust_payment_tx_history
WHERE cast(tx_datetime as date) >= '2022-08-01'
GROUP BY 1,2;
```

![ML](/images/6.ML/9.png)

We validated that the model is working. Let’s move on to generating predictions on streaming data.