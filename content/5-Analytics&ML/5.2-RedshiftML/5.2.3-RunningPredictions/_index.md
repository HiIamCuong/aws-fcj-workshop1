---
title : "Running Predictions"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 5.2.3 </b> "
---

Now our model is trained and ready to use, we can use the trained model to run predictions against the streaming data we ingested into Redshift earlier.

We can apply the transformations on top of the streaming data very easily by embedding the SQL inside the views. We will create a view on our data that aggregates our streaming data at the customer level. We will then create a second view that will aggregate data at the terminal level and then finally a third view, which will combine incoming transactional data with customer and terminal aggregated data and will call the prediction function all in one place

## Creating the views

Create the first view, which aggregates streaming data at the customer level:

```
CREATE VIEW public.customer_transformations
AS SELECT 
    customer_id, 
    CUSTOMER_ID_NB_TX_1DAY_WINDOW, 
    CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW,
    CUSTOMER_ID_NB_TX_7DAY_WINDOW,
    CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW,
    CUSTOMER_ID_NB_TX_30DAY_WINDOW, 
    CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW
FROM (
    SELECT 
        customer_id,
        SUM(CASE WHEN cast(a.TX_DATETIME AS date) = cast(getdate() AS date) THEN TX_AMOUNT  ELSE 0 end) AS CUSTOMER_ID_NB_TX_1DAY_WINDOW,
        AVG(CASE WHEN cast(a.TX_DATETIME AS date) = cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end ) AS CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW,
        SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end) AS CUSTOMER_ID_NB_TX_7DAY_WINDOW,
        AVG(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN TX_AMOUNT  ELSE 0 end ) AS CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW,
        SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -30 AND cast(getdate() AS date) THEN TX_AMOUNT ELSE 0 end) AS CUSTOMER_ID_NB_TX_30DAY_WINDOW,
        AVG( CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -30 AND cast(getdate() AS date) THEN TX_AMOUNT  ELSE 0 end ) AS CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW
    FROM (
        SELECT 
            CUSTOMER_ID, 
            TERMINAL_ID, 
            TX_AMOUNT, 
            cast(TX_DATETIME AS timestamp) TX_DATETIME
        FROM CUST_PAYMENT_TX_STREAM  --retrieve streaming data
        WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
        UNION
        SELECT 
            CUSTOMER_ID, 
            TERMINAL_ID, 
            TX_AMOUNT,
            cast(TX_DATETIME AS timestamp) TX_DATETIME
        FROM cust_payment_tx_history   -- retrieve historical data
        WHERE  cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
    ) a
GROUP BY 1
);
```

![ML](/images/6.ML/10.png)

Then create the second view, which aggregates streaming data at terminal level:

```
CREATE VIEW public.terminal_transformations
AS SELECT 
    TERMINAL_ID,
    TERMINAL_ID_NB_TX_1DAY_WINDOW, 
    TERMINAL_ID_RISK_1DAY_WINDOW, 
    TERMINAL_ID_NB_TX_7DAY_WINDOW, 
    TERMINAL_ID_RISK_7DAY_WINDOW,
    TERMINAL_ID_NB_TX_30DAY_WINDOW, 
    TERMINAL_ID_RISK_30DAY_WINDOW
FROM (
    SELECT 
    TERMINAL_ID, 
    MAX(cast(a.TX_DATETIME AS date) ) maxdt, 
    MIN(cast(a.TX_DATETIME AS date) ) mindt, 
    MAX(tx_fraud) mxtf, 
    MIN(tx_fraud) mntf, 
    SUM(CASE WHEN tx_fraud =1 THEN 1 ELSE 0 end) sumtxfraud,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -7 AND cast(getdate() AS date)  AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY1,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN tx_fraud ELSE 0 end)  AS NB_TX_DELAY1 ,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -8 AND cast(getdate() AS date)  AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY_WINDOW1,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -8 AND cast(getdate() AS date) THEN 1 ELSE 0 end)  AS NB_TX_DELAY_WINDOW1,
    NB_FRAUD_DELAY_WINDOW1-NB_FRAUD_DELAY1 AS NB_FRAUD_WINDOW1,
    NB_TX_DELAY_WINDOW1-NB_TX_DELAY1 AS TERMINAL_ID_NB_TX_1DAY_WINDOW,
    CASE WHEN TERMINAL_ID_NB_TX_1DAY_WINDOW = 0 
        THEN 0 
        ELSE NB_FRAUD_WINDOW1 / TERMINAL_ID_NB_TX_1DAY_WINDOW  
    end AS terminal_id_risk_1day_window ,--7 day
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -7 AND cast(getdate() AS date) AND tx_fraud = 1 THEN tx_fraud ELSE 0 end ) AS NB_FRAUD_DELAY7,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -7 AND cast(getdate() AS date) THEN 1 ELSE 0 end)  AS NB_TX_DELAY7,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -14 AND cast(getdate() AS date)  AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY_WINDOW7,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date) -14 AND cast(getdate() AS date) THEN 1 ELSE 0 end)  AS NB_TX_DELAY_WINDOW7,
    NB_FRAUD_DELAY_WINDOW7-NB_FRAUD_DELAY7 AS NB_FRAUD_WINDOW7,
    NB_TX_DELAY_WINDOW7-NB_TX_DELAY7 AS TERMINAL_ID_NB_TX_7DAY_WINDOW,
    CASE WHEN TERMINAL_ID_NB_TX_7DAY_WINDOW = 0 
        THEN 0 
        ELSE NB_FRAUD_WINDOW7 / TERMINAL_ID_NB_TX_7DAY_WINDOW 
    END AS terminal_id_risk_7day_window, --30 day period
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date)-7 AND cast(getdate() AS date)  AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY30,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date)-7 AND cast(getdate() AS date) THEN 1 ELSE 0 end)  AS NB_TX_DELAY30,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date)-37 AND cast(getdate() AS date)  AND tx_fraud = 1 THEN tx_fraud ELSE 0 end) AS NB_FRAUD_DELAY_WINDOW30,
    SUM(CASE WHEN cast(a.TX_DATETIME AS date) BETWEEN  cast(getdate() AS date)-37 AND cast(getdate() AS date) THEN 1 ELSE 0 end)  AS NB_TX_DELAY_WINDOW30,
    NB_FRAUD_DELAY_WINDOW30-NB_FRAUD_DELAY30 AS NB_FRAUD_WINDOW30,
    NB_TX_DELAY_WINDOW30-NB_TX_DELAY30 AS TERMINAL_ID_NB_TX_30DAY_WINDOW,
    CASE WHEN TERMINAL_ID_NB_TX_30DAY_WINDOW = 0 
        THEN 0 
        ELSE NB_FRAUD_WINDOW30 / TERMINAL_ID_NB_TX_30DAY_WINDOW 
    END AS TERMINAL_ID_RISK_30DAY_WINDOW
    FROM(
        SELECT 
            TERMINAL_ID, 
            TX_AMOUNT, 
            cast(TX_DATETIME AS timestamp) TX_DATETIME, 
            0 AS TX_FRAUD
        FROM cust_payment_tx_stream
        WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
        UNION ALL
        SELECT  
            TERMINAL_ID,
            TX_AMOUNT,
            TX_DATETIME,
            TX_FRAUD
        FROM cust_payment_tx_history  
        WHERE cast(TX_DATETIME AS date) BETWEEN cast(getdate() AS date) -37 AND cast(getdate() AS date)
    ) a
GROUP BY 1
);
```

![ML](/images/6.ML/11.png)

Create the third view, which combines incoming transactional data with customer and terminal aggregated data and calls the prediction function all in one place:

```
CREATE VIEW public.cust_payment_tx_fraud_predictions
AS SELECT
    A.APPROXIMATE_ARRIVAL_TIMESTAMP,
    D.FULL_NAME, 
    D.EMAIL_ADDRESS, 
    D.PHONE_NUMBER,
    A.TRANSACTION_ID, 
    A.TX_DATETIME, 
    A.CUSTOMER_ID, 
    A.TERMINAL_ID,
    A.TX_AMOUNT ,
    A.TX_TIME_SECONDS ,
    A.TX_TIME_DAYS ,
    fn_customer_cc_fd(
        A.TX_AMOUNT ,
        A.TX_DURING_WEEKEND,
        A.TX_DURING_NIGHT,
        C.CUSTOMER_ID_NB_TX_1DAY_WINDOW ,
        C.CUSTOMER_ID_AVG_AMOUNT_1DAY_WINDOW ,
        C.CUSTOMER_ID_NB_TX_7DAY_WINDOW ,
        C.CUSTOMER_ID_AVG_AMOUNT_7DAY_WINDOW ,
        C.CUSTOMER_ID_NB_TX_30DAY_WINDOW ,
        C.CUSTOMER_ID_AVG_AMOUNT_30DAY_WINDOW ,
        T.TERMINAL_ID_NB_TX_1DAY_WINDOW ,
        T.TERMINAL_ID_RISK_1DAY_WINDOW ,
        T.TERMINAL_ID_NB_TX_7DAY_WINDOW ,
        T.TERMINAL_ID_RISK_7DAY_WINDOW ,
        T.TERMINAL_ID_NB_TX_30DAY_WINDOW ,
        T.TERMINAL_ID_RISK_30DAY_WINDOW
    ) FRAUD_PREDICTION
FROM(
    SELECT
        APPROXIMATE_ARRIVAL_TIMESTAMP,
        TRANSACTION_ID, 
        TX_DATETIME, 
        CUSTOMER_ID, 
        TERMINAL_ID,
        TX_AMOUNT,
        TX_TIME_SECONDS,
        TX_TIME_DAYS,
        CASE WHEN extract(dow from cast(TX_DATETIME as timestamp)) in (1,7) then 1 else 0 end as TX_DURING_WEEKEND,
        CASE WHEN extract(hour from cast(TX_DATETIME as timestamp)) BETWEEN 00 and 06 then 1 else 0 end as TX_DURING_NIGHT
    FROM public.cust_payment_tx_stream
) a
JOIN public.terminal_transformations t ON a.terminal_id = t.terminal_id
JOIN public.customer_transformations c ON a.customer_id = c.customer_id
JOIN rdsmysql_zeroetl.seedingdemo.customer_info d ON a.customer_id = d.customer_id
WITH NO SCHEMA BINDING;
```

![ML](/images/6.ML/12.png)

Run a SELECT statement on the view:

```
SELECT * 
FROM cust_payment_tx_fraud_predictions
WHERE FRAUD_PREDICTION = 1;
```

![ML](/images/6.ML/13.png)

As you run the SELECT statement repeatedly, the latest credit card transactions go through transformations and ML predictions in near-real time.

Congratulations! You have done real time predictions using Redshift Zero-ETL approaches.