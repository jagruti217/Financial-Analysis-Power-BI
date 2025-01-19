# Financial-Analysis-Power-BI
This project focuses on analyzing credit card usage and financial metrics for a banking institution using Power BI and DAX queries. The goal is to provide key insights into customer behavior, credit utilization, and financial performance by applying advanced DAX functions.

## Project Overview

The analysis covers various aspects of financial data, including:

- Running totals of credit card transactions.
- 4-week moving averages of credit limits for each client.
- Month-over-month and week-over-week growth in transaction amounts.
- Customer Acquisition Cost (CAC) as a ratio of transaction amounts.
- Yearly average of client utilization ratios.
- Top 5 clients by total transaction amount.
- Credit risk scores based on multiple financial metrics.
- Churn indicators for inactive clients.

## Problem Statements and DAX Queries

1. **Running Total of Credit Card Transactions**

    **DAX Query:**
    ```DAX
    running_total = CALCULATE(SUM(credit_card[Total_Trans_Amt]),
    FILTER(ALL('credit_card'),credit_card[Week_Start_Date] <= MAX(credit_card[Week_Start_Date])))
    ```

2. **4-Week Moving Average of the Credit Limit for Each Client**

    **DAX Query:**
    ```DAX
    moving_average =
    var weeks = DATESINPERIOD('calendar'[Date], MAX('calendar'[Date]), -28, DAY)
    var sales = CALCULATE(SUM(credit_card[Credit_Limit]),weeks)
    var distinct_week = CALCULATE(DISTINCTCOUNT('calendar'[week_number]),weeks)
    RETURN DIVIDE(sales, distinct_week, 0)
    ```

3. **Month-on-Month (MoM%) and Week-on-Week (WoW%) Growth on Transaction Amount**

    **DAX Query:**
    ```DAX
    mom%growth =
    var previous_month = CALCULATE(SUM(credit_card[Total_Trans_Amt]),DATEADD('calendar'[Date], -1,MONTH))
    return DIVIDE(SUM(credit_card[Total_Trans_Amt])-previous_month,previous_month,0)

    wow%growth =
    var prev_week = CALCULATE(SUM(credit_card[Total_Trans_Amt]), DATEADD('calendar'[Date],-7,DAY))
    return DIVIDE(SUM(credit_card[Total_Trans_Amt])- prev_week, prev_week,0)
    ```

4. **Customer Acquisition Cost (CAC) as a Ratio of Transaction Amount**

    **DAX Query:**
    ```DAX
    cac_ratio = DIVIDE(
    SUM(credit_card[Customer_Acq_Cost]),
    sum(credit_card[Total_Trans_Amt])) 
    ```

5. **Yearly Average of Avg_Utilization_Ratio for All Clients**

    **DAX Query:**
    ```DAX
    avg_utilization_rate =
    AVERAGE(credit_card[Avg_Utilization_Ratio])/
    DISTINCTCOUNT(credit_card[current_year])
    ```

6. **Percentage of Interest Earned Compared to Total Revolving Balance for Each Client**

    **DAX Query:**
    ```DAX
    Interest_Earned_to_Total_Revolving_Bal = DIVIDE(
    SUM(credit_card[Interest_Earned]),
    SUM(credit_card[Total_Revolving_Bal]),0)
    ```

7. **Top 5 Clients by Total Transaction Amount**

    **DAX Query:**
    ```DAX
    top_5_clients_by_trans_amt =
    TOPN(5,SUMMARIZE(credit_card, credit_card[Client_Num],"total_amount", SUM(credit_card[Total_Trans_Amt])),[total_amount],DESC)
    ```

8. **Clients Whose Avg_Utilization_Ratio Exceeds 80%**

    **DAX Query:**
    ```DAX
    Avg_Uti_Exceeds_80% = 
    IF([Avg_Utilization_Ratio] > 0.8, TRUE, FALSE)
    ```

9. **Customer Churn Indicator (Clients with No Transactions in Last 6 Months)**

    **DAX Query:**
    ```DAX
    no_trans_in_last_6_mon = 
    var six_month = CALCULATE(SUM(credit_card[Total_Trans_Amt]), DATESINPERIOD('calendar'[Date],MAX('calendar'[Date]), -6, MONTH))
    RETURN(IF(ISBLANK(six_month), TRUE, FALSE))
    ```

10. **Delinquency Rate (Percentage of Clients with Delinquent Accounts)**

    **DAX Query:**
    ```DAX
    Delinquency_Rate_Above_0 =
    var delinuent_acc = CALCULATE(COUNTROWS(credit_card),credit_card[Delinquent_Acc] > 0)
    var total_acc = COUNTROWS(credit_card)
    RETURN DIVIDE(delinuent_acc, total_acc)
    ```

11. **Credit Risk Score (Based on Avg_Utilization_Ratio, Delinquent Accounts, and Total Revolving Balance)**

    **DAX Query:**
    ```DAX
    credit_risk_score =
    0.5*credit_card[Avg_Utilization_Ratio]+
    0.3*credit_card[Delinquent_Acc]+
    0.2*credit_card[Normalised_Revolving_Balance]
    ```

12. **Income vs Credit Limit Correlation(Created using Quick Measure)**

    **DAX Query:**
    ```DAX
    Income and Credit_Limit correlation for Client_Num = 
      VAR __CORRELATION_TABLE = VALUES('customer'[Client_Num])
      VAR __COUNT =
      	COUNTX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('customer'[Income]) * SUM('credit_card'[Credit_Limit]))
      	)
      VAR __SUM_X =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('customer'[Income]))
      	)
      VAR __SUM_Y =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('credit_card'[Credit_Limit]))
      	)
      VAR __SUM_XY =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('customer'[Income]) * SUM('credit_card'[Credit_Limit]) * 1.)
      	)
      VAR __SUM_X2 =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('customer'[Income]) ^ 2)
      	)
      VAR __SUM_Y2 =
      	SUMX(
      		KEEPFILTERS(__CORRELATION_TABLE),
      		CALCULATE(SUM('credit_card'[Credit_Limit]) ^ 2)
      	)
      RETURN
      	DIVIDE(
      		__COUNT * __SUM_XY - __SUM_X * __SUM_Y * 1.,
      		SQRT(
      			(__COUNT * __SUM_X2 - __SUM_X ^ 2)
      				* (__COUNT * __SUM_Y2 - __SUM_Y ^ 2)
      		)
      	)
    ```

13. **Average Customer Satisfaction Score by Credit Card Category**

    **DAX Query:**
    ```DAX
    avg_score_by_card_category =
    SUMMARIZE(credit_card,credit_card[Card_Category], "avg score", ROUND(AVERAGE(customer[Cust_Satisfaction_Score]),2))
    ```

14. **Loan Approval vs Credit Limit (Analyzing How Credit Limit Affects Loan Approval)**

    **DAX Query:**
    ```DAX
    loan_yes = CALCULATE(AVERAGE(credit_card[Credit_Limit]), customer[Personal_loan] = "yes")

    loan_no = CALCULATE(AVERAGE(credit_card[Credit_Limit]), customer[Personal_loan] = "no")
    ```

15. **High Risk Clients Flag (Clients with Revolving Balance > 90% of Credit Limit and High Avg_Utilization_Ratio)**

    **DAX Query:**
    ```DAX
    exceeds_90%_credit_limit =
    var clAbove90 = credit_card[Credit_Limit] * 0.9
    RETURN IF(credit_card[Total_Revolving_Bal] > clAbove90 && [Avg_Utilization_Ratio] > 0.5, True,False)
    ```

## Tools Used

- **Power BI**: Data visualization and reporting.
- **DAX**: Data Analysis Expressions for financial metrics calculation.

## Getting Started

1. Download the Power BI file containing the financial analysis.
2. Review the visuals, KPIs, and calculated metrics based on DAX queries.
3. Explore customer behavior and financial metrics via interactive dashboards.
