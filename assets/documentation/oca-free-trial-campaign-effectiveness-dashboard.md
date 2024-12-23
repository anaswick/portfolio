# Free Trial Campaign Effectiveness Dashboard

### Tools used : PostgreSQL, Metabase

## Free Trial Campaign Metrics

![image](https://github.com/user-attachments/assets/53b86948-dc38-4859-9fe2-5c6b433a496c)

**Purpose** <br>
This section provides key metrics to evaluate the effectiveness of the free trial campaign. The metrics include total account creation, active users on the free trial (segmented by Blast Lite and Interaction Lite), and the conversion rate from free trial to paid users. These metrics help gauge user engagement and the success of the free trial in driving paid conversions.

**Usage** <br>
Total Account Created: Tracks the total number of new accounts created through the free trial campaign, representing initial user acquisition.
Active User Free Trial - Blast Lite: Measures the number of users actively using the "Blast Lite" feature during their trial, indicating engagement with core functionality.
Active User Free Trial - Interaction Lite: Measures active users on the "Interaction Lite" feature, showing how many trial users engage with this aspect of the product.
Free Trial Conversion Rate: Calculates the percentage of free trial users who converted to paid subscriptions, providing insight into the trial’s effectiveness in generating long-term customers.
These metrics allow stakeholders to monitor trial performance across user acquisition, engagement, and conversion.

**Impact** <br>
Understanding these metrics enables the organization to optimize the free trial experience by identifying areas for improvement in user engagement and conversion. By analyzing trends in active usage and conversion rates, stakeholders can make data-driven decisions to refine campaign strategies, enhance the user journey, and ultimately increase revenue through successful free-to-paid conversions. This data-driven approach helps align the free trial campaign with broader business objectives, maximizing the return on investment for user acquisition efforts.

## Daily Registered Free Trial Users

![image](https://github.com/user-attachments/assets/762d0a00-246c-4f01-b577-95e5f4ca405f)

**Purpose** <br>
This line chart tracks the daily count of new users registering for the free trial. It provides insights into user acquisition trends, highlighting fluctuations in daily registrations that can indicate the effectiveness of marketing efforts or external factors influencing sign-ups.

**Usage** <br>
The x-axis represents the date, while the y-axis shows the number of new free trial registrations each day. Peaks and dips in the chart reflect days with higher or lower sign-ups, allowing stakeholders to observe daily trends. This information can be used to assess the impact of promotional activities, advertising campaigns, or seasonal variations on trial registrations.

This visualization enables stakeholders to quickly identify spikes or drops in daily registrations, providing valuable context for understanding user acquisition performance over time.

**Impact** <br>
Analyzing daily free trial registrations helps the organization optimize marketing and outreach strategies to increase sign-ups. By identifying patterns in registration trends, stakeholders can adjust campaign timing, target specific periods of high activity, and evaluate the impact of marketing initiatives on user acquisition. This data-driven approach enhances decision-making for user growth strategies and aligns trial sign-up efforts with broader business goals for customer acquisition.


**Datamart Query**
```
truncate mart.free_trial_users ;

insert into mart.free_trial_users 
select 
b."_id" as billingmasters_id,
b.application , 
u.full_name , 
b.account_status , 
b.billing_type , 
case when b.is_new = true then 'lite'
else 'reguler' end as account_category,
b.user_type ,
b.product,
a.created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta' AS account_created_at,
b.created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta' AS transaction_ts,
b.billing_date AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta' AS billing_date,
b.next_billing_date AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta' AS next_billing_date,
b.subscription_type ,
b.previous_type 
from raw.billingmasters b 
left join raw.applications a on b.application = a."_id" 
left join raw.users u on a."user" = u."_id" 
order by account_created_at;
```

**Metabase Query**
```
select DATE(account_created_at) AS date, count(distinct application)
from mart.free_trial_users
where {{account_created_at}} and application not in (select application from raw.master_testing_demo_account) and {{subscription_type}}
group by 1;
```

## Free Trial Favorite Message Blasting Channel

![image](https://github.com/user-attachments/assets/b066b2f2-0741-4e30-86e2-1d35c82761aa)

**Purpose** <br>
This bar chart shows the distribution of message blasts sent through different communication channels during the free trial period. It highlights which channels are primarily used to engage users, helping to assess the effectiveness of each channel in reaching trial users.

**Usage** <br>
The x-axis lists the different channels used for message blasts, including WhatsApp Utility (wa_utility), WhatsApp Marketing (wa_marketing), email, and SMS. The y-axis represents the total number of blasts sent. In this chart, WhatsApp Marketing is the predominant channel used, with a significantly higher blast count compared to other channels.

This visualization helps stakeholders quickly understand which channels are most actively utilized for engaging free trial users, providing insight into channel-specific usage.

**Impact** <br>
By analyzing channel distribution for message blasts, the organization can evaluate the effectiveness of each communication channel in driving engagement during the free trial period. This data enables informed decisions on optimizing channel strategies, potentially shifting focus to high-performing channels or exploring underutilized ones. Understanding channel effectiveness supports targeted marketing efforts, enhances user engagement, and helps improve conversion rates from free trial to paid subscriptions.

**Datamart Query**
```
truncate mart.free_trial_blast_users ;

insert into mart.free_trial_blast_users
SELECT 
ftu.billingmasters_id,
ftu.application,
ftu.full_name,
ftu.account_status,
ftu.billing_type,
ftu.account_category,
ftu.user_type,
ftu.account_created_at,
ftu.transaction_ts,
ftu.billing_date,
ftu.next_billing_date,
ftu.subscription_type,
    SUM(wbta.marketing) AS marketing_count,
    SUM(wbta.utility) AS utility_count,
    SUM(ebta.total) AS email_count,
    SUM(sbta.total_sms) AS sms_count,
    CASE 
        WHEN SUM(wbta.marketing) IS NOT NULL 
            OR SUM(wbta.utility) IS NOT NULL 
            OR SUM(ebta.total) IS NOT NULL 
            OR SUM(sbta.total_sms) IS NOT NULL 
        THEN 'active'
        ELSE 'inactive'
    END AS user_status_blast,
ftu.previous_type ,
NOW() _ingest_ts
FROM mart.free_trial_users ftu
LEFT JOIN mart.wa_blast_transaction_aggregate wbta ON ftu.full_name = wbta.client_name 
LEFT JOIN mart.email_blast_transaction_aggregate ebta ON ftu.application = ebta.application 
LEFT JOIN mart.sms_blast_transaction_aggregate sbta ON ftu.application = sbta.application 
WHERE ftu.application NOT IN (SELECT application FROM raw.master_testing_demo_account)
GROUP BY 
	ftu.billingmasters_id,
    ftu.application, 
    ftu.full_name, 
    ftu.account_status, 
    ftu.billing_type, 
    ftu.account_category, 
    ftu.user_type, 
    ftu.account_created_at, 
    ftu.transaction_ts, 
    ftu.billing_date, 
    ftu.next_billing_date, 
    ftu.subscription_type,
    ftu.previous_type 
ORDER BY ftu.account_created_at;
```
**Metabase Query**
```
SELECT 'wa_marketing' AS category, SUM(marketing_count) AS total
FROM mart.free_trial_blast_users
WHERE {{date}} AND {{subscription_type}}

UNION ALL

SELECT 'wa_utility' AS category, SUM(utility_count) AS total
FROM mart.free_trial_blast_users
WHERE {{date}} AND {{subscription_type}}

UNION ALL

SELECT 'email' AS category, SUM(email_count) AS total
FROM mart.free_trial_blast_users
WHERE {{date}} AND {{subscription_type}}

UNION ALL

SELECT 'sms' AS category, SUM(sms_count) AS total
FROM mart.free_trial_blast_users
WHERE {{date}} AND {{subscription_type}}

ORDER BY total asc, category asc;
```

## Retention Analysis - Daily Cohort

![image](https://github.com/user-attachments/assets/d09d7f45-51b6-4f47-bc1d-0dff26a4587c)

**Purpose** <br>
This heatmap tracks user retention on a day-by-day basis, segmented by cohorts. Each cohort represents users who signed up on a specific date, and the chart shows the percentage of users from each cohort who return each day over a 14-day period. This visualization provides insights into user engagement and retention trends during the early stages of a user's journey.

**Usage** <br>
Cohortday: The leftmost column lists the dates when each cohort (group of users) signed up.
Cohort Size: Indicates the number of users in each cohort.
Day-by-Day Retention: Each subsequent column (day_0 to day_14) shows the percentage of users from each cohort who returned on that specific day. Cells are color-coded, with green representing higher retention and red indicating lower retention.
For example, if a cell shows “50%” in the day_1 column, it means that 50% of users from that cohort returned the day after signing up. This chart enables stakeholders to observe daily retention patterns and identify how quickly engagement drops off for each cohort.

**Impact** <br>
By analyzing cohort retention data, the organization can identify which days experience significant drop-offs and understand the overall retention pattern during the free trial period. This information helps in tailoring engagement strategies, such as sending targeted reminders or offering incentives to encourage users to return. Improving early retention rates can lead to higher conversions from free trial to paid users, as engaged users are more likely to see the value in the service. Ultimately, this data-driven approach to retention enhances user experience, increases the likelihood of long-term retention, and supports sustainable growth.

**Metabase Query**
```
SELECT 
    cohortday,
    COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END) AS cohort_size, -- Initial cohort size
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_0,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 1 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_1,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 2 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_2,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 3 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_3,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 4 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_4,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 5 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_5,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 6 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_6,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 7 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_7,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 8 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_8,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 9 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_9,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 10 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_10,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 11 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_11,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 12 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_12,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 13 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_13,
    ROUND(
        COUNT(DISTINCT CASE WHEN cohort = 14 THEN application_id END) * 100.0 / NULLIF(COUNT(DISTINCT CASE WHEN cohort = 0 THEN application_id END), 0), 2
    ) AS day_14
FROM 
    (WITH ranked_history_logs AS (
        SELECT 
            hl."_dataid",
            hl."_id" AS transaction_id,
            hl.application AS application_id,
            hl.category,
            hl.created_at AS trans_ts,
            hl."_ingest_ts",
            ROW_NUMBER() OVER (PARTITION BY hl."_id" ORDER BY hl."_ingest_ts" DESC) AS row_num,
            DATE_TRUNC('day', hl.created_at) AS transday, -- Extract transaction day
            MIN(hl.created_at) OVER (PARTITION BY hl.application) AS first_trans_ts -- First transaction per application_id
        FROM 
            raw.history_logs hl
        JOIN 
            raw.applications a ON hl.application = a."_id"
    )
    SELECT 
        application_id,
        transaction_id,
        trans_ts,
        category,
        transday,
        DATE_TRUNC('day', first_trans_ts) AS cohortday, -- Truncate to day for cohort day
        DATE_PART('day', AGE(transday, DATE_TRUNC('day', first_trans_ts))) AS cohort -- Calculate day difference between cohortday and transday
    FROM 
        ranked_history_logs
    WHERE 
        row_num = 1 
        AND application_id NOT IN (SELECT application FROM raw.master_testing_demo_account mtda) 
        AND first_trans_ts >= '2024-11-01' -- Start from 2024-11-01
    ORDER BY 
        application_id, trans_ts ASC) cohort_analysis
GROUP BY 
    cohortday
ORDER BY 
    cohortday;
```

## Free Trial Blast - Account Creation to First Login Interval

![image](https://github.com/user-attachments/assets/24cf5955-10e6-49e4-9bb3-b869694f98bf)

**Purpose** <br>
This section measures the time interval from account creation to first login for users on the free trial. By tracking the time it takes for new users to log in after signing up, this metric provides insights into user onboarding efficiency and initial engagement with the platform.

**Usage** <br>
The metrics displayed include:

**Minimum Interval**: The shortest time taken by a user to log in after account creation, indicating the quickest engagement. <br>
**Average Interval**: The average time across all users from account creation to first login, showing the typical onboarding duration. <br>
**Maximum Interval**: The longest time a user took to log in after account creation, highlighting potential delays in initial engagement. <br>

These metrics allow stakeholders to assess the effectiveness of the onboarding process, identifying how quickly new users engage with the platform after signing up.

**Impact** <br>
Understanding the time interval from account creation to first login helps the organization improve the onboarding experience. A shorter interval suggests effective onboarding and immediate user interest, while longer intervals may indicate friction points in the process. This information enables the team to refine onboarding strategies, such as sending timely reminders or providing introductory tutorials, to encourage faster engagement. Ultimately, enhancing early engagement can lead to higher conversion rates and improved user satisfaction, supporting the success of the free trial campaign.


## Free Trial Blast - Account Creation to First Broadcast Interval

![image](https://github.com/user-attachments/assets/0c20a7db-52d1-4f51-b10e-6b5a3109df07)

**Purpose** <br>
This section measures the time interval from account creation to the first broadcast for users on the free trial. Tracking how quickly users initiate their first broadcast after signing up provides insights into the initial engagement and ease of using the broadcast feature during the trial period.

**Usage** <br>
The metrics displayed include:

**Minimum Interval**: The shortest time taken by a user to start their first broadcast after account creation, showing the quickest initial engagement. <br>
**Average Interval**: The average time across all users from account creation to first broadcast, reflecting typical user behavior during the onboarding process. <br>
**Maximum Interval**: The longest time a user took to initiate their first broadcast after account creation, indicating potential delays in feature exploration. <br>

These metrics enable stakeholders to assess how readily new users engage with the broadcast feature, offering a view into the onboarding effectiveness for this core functionality.

**Impact** <br>
Understanding the time interval from account creation to the first broadcast helps identify potential barriers to initial engagement with the platform’s key feature. A shorter interval suggests effective onboarding and immediate value realization by users, while longer intervals may indicate areas where additional guidance or feature prompts are needed. By optimizing this interval, the organization can improve user onboarding, enhance the trial experience, and increase the likelihood of converting trial users to paying customers. This approach supports better user engagement and ultimately contributes to campaign success and customer retention.
