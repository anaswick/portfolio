# Smart Segmentation: Machine Learning and RFM for Better Customer Relationships

### Tools used : PostgreSQL, Python, Excel

In this project, my team developed a machine learning-powered RFM segmentation framework to enhance customer relationship strategies for an omnichannel message blasting platform. The framework leverages customer transaction data to calculate Recency, Frequency, and Monetary (RFM) scores, enabling the identification of key customer segments across multiple communication channels, including SMS, email, WhatsApp, and social media.

Key features include automated segmentation using clustering algorithms, predictive insights into customer behavior, and actionable recommendations for personalized marketing strategies. The framework is integrated into an interactive dashboard, providing stakeholders with real-time visualizations of customer engagement, retention risks, and revenue opportunities.

This project directly impacted the business by improving targeting precision, reducing churn rates, and increasing campaign ROI. By identifying high-value customers and tailoring engagement strategies, the framework enhanced customer lifetime engagement and value. Additionally, the data-driven insights empowered the team to make strategic decisions, optimize channel performance, and foster stronger, long-term customer relationships.

This project demonstrates my ability to apply machine learning and analytics to drive measurable business outcomes, create customer-centric strategies, and unlock growth opportunities in a complex omnichannel environment.

## 1. Data Preparation

**Dataset 1 : Recency**

Preparing the first datasets for recency, here is the PostgreSQL query :
```
SELECT 'wa' AS source, 
       application, 
       MAX(created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') AS most_recent_campaign
FROM raw.whatsappbroadcasts
WHERE status = 'finish' 
  AND DATE_PART('year', created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') = 2024
GROUP BY application
UNION ALL
SELECT 'sms' AS source, 
       application, 
       MAX(created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') AS most_recent_campaign
FROM raw.smsbroadcasts
WHERE status = 'finish' 
  AND DATE_PART('year', created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') = 2024
GROUP BY application
UNION ALL
SELECT 'email' AS source, 
       application, 
       MAX(created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') AS most_recent_campaign
FROM raw.emailbroadcasts
WHERE status = 'finish' 
  AND DATE_PART('year', created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') = 2024
GROUP BY application
UNION ALL
SELECT 'call' AS source, 
       application, 
       MAX(created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') AS most_recent_campaign
FROM raw.broadcastcalls
WHERE status = 'finish' 
  AND DATE_PART('year', created_at AT TIME ZONE 'UTC' AT TIME ZONE 'Asia/Jakarta') = 2024
GROUP BY application;

```

Dataset Preview

<img width="1209" alt="Screenshot 2024-12-16 at 09 53 18" src="https://github.com/user-attachments/assets/7d827f47-ba27-4c5e-bb68-750a5b71fd4c" />


**Dataset 2 : Frequency by The Number of Campaign Executed**

Preparing the second dataset for frequency by the number of campaign executed, here is the query :

```
WITH wa_broadcast AS (
    SELECT 
        application, 
        COUNT(DISTINCT "_id") AS wa_broadcast_count
    FROM 
        raw.whatsappbroadcasts w
    WHERE 
        status = 'finish'
        AND created_at BETWEEN '2024-01-01 00:00:00' AND '2024-11-30 23:59:59' -- Filter by date range
    GROUP BY 
        application
),
sms_broadcast AS (
    SELECT 
        application, 
        COUNT(DISTINCT "_id") AS sms_broadcast_count
    FROM 
        raw.smsbroadcasts s
    WHERE 
        status = 'finish'
        AND created_at BETWEEN '2024-01-01 00:00:00' AND '2024-11-30 23:59:59' -- Filter by date range
    GROUP BY 
        application
),
email_broadcast AS (
    SELECT 
        application, 
        COUNT(DISTINCT "_id") AS email_broadcast_count
    FROM 
        raw.emailbroadcasts e
    WHERE 
        status = 'finish'
        AND created_at BETWEEN '2024-01-01 00:00:00' AND '2024-11-30 23:59:59' -- Filter by date range
    GROUP BY 
        application
)
SELECT 
    COALESCE(wa.application, sms.application, email.application) AS application,
    COALESCE(wa.wa_broadcast_count, 0) AS wa_broadcast_count,
    COALESCE(sms.sms_broadcast_count, 0) AS sms_broadcast_count,
    COALESCE(email.email_broadcast_count, 0) AS email_broadcast_count
FROM 
    wa_broadcast wa
FULL OUTER JOIN 
    sms_broadcast sms
ON 
    wa.application = sms.application
FULL OUTER JOIN 
    email_broadcast email
ON 
    COALESCE(wa.application, sms.application) = email.application
ORDER BY 
    application;
```

Dataset Preview
<img width="866" alt="Screenshot 2024-12-16 at 09 50 35" src="https://github.com/user-attachments/assets/7f4af596-7829-4471-826e-4539ce2da827" />


**Dataset 3 : Frequency and Monetary by The Number of Charged Message**

Preparing the last dataset for frequency and monetary by the number of charged message. For the context, we only charged sent message, here is the query : 

```
WITH wa_blast AS (
    SELECT 
        application,
        client_name AS full_name,
        wabanumber,
        SUM(marketing) AS total_marketing,
        SUM(utility) AS total_utility,
        SUM(service) AS total_service,
        SUM(authentication) AS total_authentication,
        SUM(marketing * 777) AS rev_marketing,
        SUM(utility * 400) AS rev_utility,
        SUM(service * 382) AS rev_service,
        SUM(authentication * 579) AS rev_authentication
    FROM 
        mart.wa_blast_transaction_aggregate
    GROUP BY 
        application, client_name, wabanumber
),
sms_blast AS (
    SELECT 
        application,
        full_name,
        SUM(total_sms) AS total_sms,
        SUM(total_sms * 600) AS rev_sms
    FROM 
        mart.sms_blast_transaction_aggregate
    GROUP BY 
        application, full_name
),
email_blast AS (
    SELECT 
        application,
        full_name,
        SUM(total) AS total_email,
        SUM(total * 23) AS rev_email
    FROM 
        mart.email_blast_transaction_aggregate
    GROUP BY 
        application, full_name
),
-- Aggregate WA data at the `application` and `full_name` level
wa_blast_aggregated AS (
    SELECT
        application,
        full_name,
        SUM(total_marketing) AS total_marketing,
        SUM(total_utility) AS total_utility,
        SUM(total_service) AS total_service,
        SUM(total_authentication) AS total_authentication,
        SUM(rev_marketing) AS rev_marketing,
        SUM(rev_utility) AS rev_utility,
        SUM(rev_service) AS rev_service,
        SUM(rev_authentication) AS rev_authentication
    FROM
        wa_blast
    GROUP BY
        application, full_name
)
-- Final join and aggregation
SELECT 
    wa.application,
    wa.full_name,
    COALESCE(wa.total_marketing, 0) AS total_marketing,
    COALESCE(wa.total_utility, 0) AS total_utility,
    COALESCE(wa.total_service, 0) AS total_service,
    COALESCE(wa.total_authentication, 0) AS total_authentication,
    COALESCE(sms.total_sms, 0) AS total_sms,
    COALESCE(email.total_email, 0) AS total_email,
    COALESCE(wa.rev_marketing, 0) AS rev_marketing,
    COALESCE(wa.rev_utility, 0) AS rev_utility,
    COALESCE(wa.rev_service, 0) AS rev_service,
    COALESCE(wa.rev_authentication, 0) AS rev_authentication,
    COALESCE(sms.rev_sms, 0) AS rev_sms,
    COALESCE(email.rev_email, 0) AS rev_email
FROM 
    wa_blast_aggregated wa
LEFT JOIN 
    sms_blast sms
ON 
    wa.application = sms.application AND wa.full_name = sms.full_name
LEFT JOIN 
    email_blast email
ON 
    wa.application = email.application AND wa.full_name = email.full_name
where wa.application not in (select application from raw.master_testing_demo_account mtda)
ORDER BY 
    wa.full_name, wa.application;
```

Dataset Preview

<img width="1220" alt="Screenshot 2024-12-16 at 09 54 59" src="https://github.com/user-attachments/assets/f524cda6-54c5-40d0-8d06-14ae4a9c818f" />

## 2. Importing Dataset to Python

### Import Library Needed for Analysis

```
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

### Import Dataset : Recency

```
df_recency =pd.read_csv('RFM_recency_all_channel.csv')
```

**Check the datatype and convert it if needed**

```
df_recency.dtypes
```

Result

<img width="301" alt="Screenshot 2024-12-16 at 10 03 58" src="https://github.com/user-attachments/assets/8ff0f113-017d-49c8-8189-d1792142b075" />

Convert the object type field into datetime UTC+7

```
# Convert date columns from object to datetime and adjust timezone
date_columns = ['wa_max_date', 'sms_max_date', 'email_max_date', 'call_max_date', 'max_all']

# Convert to datetime and adjust timezone
for col in date_columns:
    df_recency[col] = pd.to_datetime(df_recency[col], errors='coerce')  # Convert to datetime
    df_recency[col] = df_recency[col].dt.tz_localize('UTC').dt.tz_convert('Asia/Jakarta')  # Adjust to UTC+7

# Display the updated data types
print(df_recency.dtypes)

# Optional: If you want to remove the timezone information and keep it as naive datetime:
for col in date_columns:
    df_recency[col] = df_recency[col].dt.tz_localize(None)  # Remove timezone
```

<img width="378" alt="Screenshot 2024-12-16 at 10 05 34" src="https://github.com/user-attachments/assets/0e8c7dd7-9588-4e7c-aa0d-43aa1155f474" />

## Import Dataset : Frequency and Monetary by Number of Charged Message

```
df_freq_mon =pd.read_csv('RFM_frequency_total_charge.csv')
```

**Create a new column 'total_charged'**

```
# Create a new column 'total_charged' by summing up all the charge columns
df_freq_mon['total_charged'] = (
    df_freq_mon['total_marketing'] +
    df_freq_mon['total_utility'] +
    df_freq_mon['total_sms'] +
    df_freq_mon['total_email']
)

# Display the updated DataFrame
print(df_freq_mon[['application', 'total_charged']].head())
```

**Creata a new column 'total_revenue'**

```
# Create a new column 'total_revenue' by summing up all the revenue columns
df_freq_mon['total_revenue'] = (
    df_freq_mon['rev_marketing'] +
    df_freq_mon['rev_utility'] +
    df_freq_mon['rev_sms'] +
    df_freq_mon['rev_email']
)

# Display the updated DataFrame
print(df_freq_mon[['application', 'total_revenue']].head())
```

## Import Dataset : Frequency by the Number of Campaign Executed

```
df_freq_broadcast =pd.read_csv('rfm_frequency_broadcast_202401_202409.csv')
```

Create a new column 'total_broadcasts'

```
# Create a new column 'total_broadcast' by summing up all the broadcast columns
df_freq_broadcast['total_broadcast'] = (
    df_freq_broadcast['wa_broadcast_count'] +
    df_freq_broadcast['sms_broadcast_count'] +
    df_freq_broadcast['email_broadcast_count'] 
)

# Display the updated DataFrame
print(df_freq_broadcast[['application', 'total_broadcast']].head())
```

## Merge All Datasets

```
# Step 1: Merge df_recency and df_freq_mon
df_temp = pd.merge(
    df_recency,
    df_freq_mon,
    on='application',  # Joining key
    how='inner'  # Choose 'inner', 'left', 'right', or 'outer' depending on your needs
)

# Step 2: Merge the result with df_freq_broadcast
df_combined = pd.merge(
    df_temp,
    df_freq_broadcast,
    on='application',  # Joining key
    how='inner'  # Adjust the join type as needed
)

# Display the first few rows of the combined DataFrame
print(df_combined.head())

# Save the combined DataFrame to a CSV file (optional)
df_combined.to_csv('combined_rfm_data.csv', index=False)
print("Combined dataset saved to 'combined_rfm_data.csv'")
```

**Take only needed column for analysis**

```
df_combined[['application','full_name','max_all','total_revenue','total_charged','total_broadcast']]
```

**Check null value**

```
print(df_combined.isnull().sum())
```

<img width="251" alt="Screenshot 2024-12-16 at 10 13 54" src="https://github.com/user-attachments/assets/fbc8a265-f57f-4023-ae2e-0ad8b5adc78b" />


**Count the recency in days**

```
from datetime import datetime

# Convert 'max_all' column to datetime format (if not already)
df_combined['max_all'] = pd.to_datetime(df_combined['max_all'])

# Calculate today's date
today = pd.Timestamp.now()

# Calculate recency as the difference in days between today and 'max_all'
df_combined['recency_days'] = (today - df_combined['max_all']).dt.days

# Display the updated DataFrame
print(df_combined[['application','full_name','max_all', 'recency_days']].head())

# Optional: Save the updated DataFrame to a CSV file
df_combined.to_csv('combined_rfm_with_recency.csv', index=False)
print("Updated dataset with recency saved to 'combined_rfm_with_recency.csv'")
```

<img width="135" alt="Screenshot 2024-12-16 at 10 15 10" src="https://github.com/user-attachments/assets/f08f4e6f-d29c-44d7-9767-4e6b321379a5" />

**Append the new column to the existing dataset**

```
df_rfm = df_combined[['application','full_name','recency_days','total_broadcast','total_charged','total_revenue']]
df_rfm.head()
```

**Rename the column name for better clarity**

```
# Rename columns in the DataFrame
df_rfm = df_rfm.rename(columns={
    'recency_days': 'recency',
    'total_broadcast': 'frequency_broadcast',
    'total_charged' : 'frequency_charged',
    'total_revenue': 'monetary'
})

# Display the updated DataFrame
print(df_rfm.head())
```

## Scale the numerical data

```
from sklearn.preprocessing import StandardScaler
# Select only the numerical columns (exclude 'application')
numerical_data = df_rfm[['recency', 'frequency_broadcast','frequency_charged', 'monetary']]

# Initialize the scaler
scaler = StandardScaler()

# Fit and transform the numerical data
scaled_data = scaler.fit_transform(numerical_data)

# Convert the scaled data back to a DataFrame
scaled_df = pd.DataFrame(scaled_data, columns=numerical_data.columns)

# Reattach the 'application' column for reference
scaled_df['application'] = df_rfm['application']

# Display the scaled data
print(scaled_df.head())

```

## Train the datasets for RFM Clustering

```
import numpy as np
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# Use only the scaled numerical columns for clustering
scaled_numerical_data = scaled_df[['recency', 'frequency_broadcast','frequency_charged', 'monetary']]

# Initialize a list to store inertia values
inertia = []

# Iterate over a range of cluster numbers
for i in np.arange(1, 11):
    kmeans = KMeans(n_clusters=i, random_state=42)  # Set random_state for reproducibility
    kmeans.fit(scaled_numerical_data)  # Fit on numerical data
    inertia.append(kmeans.inertia_)  # Append inertia value

# Plot the inertia values to visualize the Elbow Method
plt.figure(figsize=(12, 8))
plt.plot(range(1, 11), inertia, marker="o")
plt.title("Elbow Method to Determine Optimal Clusters", fontsize=16)
plt.xlabel("Number of Clusters", fontsize=12)
plt.ylabel("Inertia", fontsize=12)
plt.grid(alpha=0.5)
plt.show()
```

**Using Elbow Method for Optimum Number of Clusters**

<img width="703" alt="Screenshot 2024-12-16 at 10 17 29" src="https://github.com/user-attachments/assets/b71f5947-32b0-419f-a029-51b415cb4777" />


```
from sklearn.cluster import KMeans

# Fit KMeans with 4 clusters
kmeans = KMeans(n_clusters=4, random_state=42)  # random_state for reproducibility
kmeans.fit(scaled_numerical_data)

# Assign cluster labels to the original DataFrame
df_rfm['Clusters'] = (kmeans.labels_ + 1)  # Shift labels to start from 1

# Display the DataFrame with the new Clusters column
print(df_rfm.head())
```


## Visualizing the clustering result

```
import matplotlib.pyplot as plt

# Count the number of customers in each cluster
cluster_counts = df_rfm['Clusters'].value_counts().sort_index()

# Create a bar chart
plt.figure(figsize=(10, 6))
plt.bar(cluster_counts.index, cluster_counts.values, color='skyblue', edgecolor='black', alpha=0.7)
plt.title('Number of Customers in Each Cluster', fontsize=16)
plt.xlabel('Cluster', fontsize=12)
plt.ylabel('Count', fontsize=12)
plt.xticks(cluster_counts.index)
plt.grid(axis='y', alpha=0.5)

# Show the bar chart
plt.tight_layout()
plt.show()
```

<img width="708" alt="Screenshot 2024-12-16 at 10 19 08" src="https://github.com/user-attachments/assets/f631b538-dc3d-477f-b82f-af6610d68320" />


```
# Median values for each cluster
cluster_medians = df_rfm.groupby('Clusters')[['recency', 'frequency_broadcast', 'frequency_charged', 'monetary']].median()

# Recency
plt.figure(figsize=(8, 6))
bars = plt.bar(cluster_medians.index, cluster_medians['recency'], color='skyblue', edgecolor='black')
plt.title('Median Recency by Cluster', fontsize=16)
plt.xlabel('Cluster', fontsize=12)
plt.ylabel('Median Recency', fontsize=12)
plt.grid(axis='y', alpha=0.5)

# Add bar labels
for bar in bars:
    plt.text(bar.get_x() + bar.get_width() / 2, bar.get_height(),
             f'{bar.get_height():.0f}', ha='center', va='bottom', fontsize=10)

plt.tight_layout()
plt.show()

# Frequency Broadcasts
plt.figure(figsize=(8, 6))
bars = plt.bar(cluster_medians.index, cluster_medians['frequency_broadcast'], color='orange', edgecolor='black')
plt.title('Median Frequency Broadcast by Cluster', fontsize=16)
plt.xlabel('Cluster', fontsize=12)
plt.ylabel('Median Frequency', fontsize=12)
plt.grid(axis='y', alpha=0.5)

# Add bar labels
for bar in bars:
    plt.text(bar.get_x() + bar.get_width() / 2, bar.get_height(),
             f'{bar.get_height():.0f}', ha='center', va='bottom', fontsize=10)

plt.tight_layout()
plt.show()

# Frequency Charged
plt.figure(figsize=(8, 6))
bars = plt.bar(cluster_medians.index, cluster_medians['frequency_charged'], color='orange', edgecolor='black')
plt.title('Median Frequency Charged by Cluster', fontsize=16)
plt.xlabel('Cluster', fontsize=12)
plt.ylabel('Median Frequency', fontsize=12)
plt.grid(axis='y', alpha=0.5)

# Add bar labels
for bar in bars:
    plt.text(bar.get_x() + bar.get_width() / 2, bar.get_height(),
             f'{bar.get_height():.0f}', ha='center', va='bottom', fontsize=10)

plt.tight_layout()
plt.show()

# Monetary
plt.figure(figsize=(8, 6))
bars = plt.bar(cluster_medians.index, cluster_medians['monetary'], color='green', edgecolor='black')
plt.title('Median Monetary by Cluster', fontsize=16)
plt.xlabel('Cluster', fontsize=12)
plt.ylabel('Median Monetary', fontsize=12)
plt.grid(axis='y', alpha=0.5)

# Add bar labels
for bar in bars:
    plt.text(bar.get_x() + bar.get_width() / 2, bar.get_height(),
             f'{bar.get_height():.2e}', ha='center', va='bottom', fontsize=10)  # Scientific notation for monetary values

plt.tight_layout()
plt.show()
```

**Median Recency by Cluster**

<img width="699" alt="Screenshot 2024-12-16 at 10 20 03" src="https://github.com/user-attachments/assets/42ce315f-ebd6-4b2d-9d91-580b496e628c" />


**Median Frequency Broadcast by Cluster**

<img width="701" alt="Screenshot 2024-12-16 at 10 20 18" src="https://github.com/user-attachments/assets/b5e6f9a4-fea0-4dc6-9f15-4e31e9f85fe9" />


**Median Frequency Charged by Cluster**

<img width="706" alt="Screenshot 2024-12-16 at 10 20 25" src="https://github.com/user-attachments/assets/7c1554b3-cdd4-4893-a843-2bf7d9d4a6e4" />


**Median Monetary by Cluster**

<img width="706" alt="Screenshot 2024-12-16 at 10 20 32" src="https://github.com/user-attachments/assets/cf38e5db-9211-47bb-bcf2-e85a8737ac88" />

```
# Count customers in each cluster
cluster_counts = df_rfm['Clusters'].value_counts()

# Bar plot for cluster counts
plt.figure(figsize=(8, 5))
sns.barplot(x=cluster_counts.index, y=cluster_counts.values, palette='viridis')
plt.title('Number of Customers in Each Cluster')
plt.xlabel('Cluster')
plt.ylabel('Number of Customers')
plt.show()

# Boxplots for each RFM metric by cluster
for col in ['recency', 'frequency_broadcast','frequency_charged', 'monetary']:
    plt.figure(figsize=(10, 6))
    sns.boxplot(x='Clusters', y=col, data=df_rfm, palette='viridis')
    plt.title(f'{col.capitalize()} by Cluster')
    plt.xlabel('Cluster')
    plt.ylabel(col.capitalize())
    plt.show()
```

**Boxplots for data distribution check**

<img width="705" alt="Screenshot 2024-12-16 at 10 24 00" src="https://github.com/user-attachments/assets/828c8ce0-f672-401c-986a-03d53cdd856b" />

<img width="704" alt="Screenshot 2024-12-16 at 10 24 07" src="https://github.com/user-attachments/assets/d54a4d70-6ecb-4c42-ad53-70942e608dcf" />

<img width="703" alt="Screenshot 2024-12-16 at 10 24 20" src="https://github.com/user-attachments/assets/2c8edf17-ca26-4901-b478-f2274a22083a" />

<img width="713" alt="Screenshot 2024-12-16 at 10 24 25" src="https://github.com/user-attachments/assets/d61b0bde-2e02-4292-9d17-5604f58ad05f" />

**Renaming each clusters for better clarity**

```
# Define cluster names based on the given descriptions
cluster_names = {
    1: "Moderately Engaged Users",
    2: "Top Performers",
    3: "High Value Frequent Users",
    4: "Dormant or Low-Value Users"
}

# Map the cluster numbers to their respective names
df_rfm['cluster_name'] = df_rfm['Clusters'].map(cluster_names)

# Create a single table with cluster information and names
cluster_table = df_rfm[['application', 'full_name', 'recency', 'frequency_broadcast','frequency_charged', 'monetary', 'Clusters', 'cluster_name']]

# Display the first few rows of the updated table
print(cluster_table.head())

# Save the table to a CSV file
cluster_table.to_csv('rfm_clusters_multidimension_with_names_mkt_uti.csv', index=False)
print("RFM clusters table with names saved to 'rfm_clusters_multidimension_with_names_mkt_uti.csv'")
```

## Revenue Contribution by Cluster

<img width="673" alt="Screenshot 2024-12-16 at 10 26 11" src="https://github.com/user-attachments/assets/10f02f7e-6fcd-470e-b99e-b70484a1be5d" />


## RFM Analysis Interpretation

# RFM Cluster Analysis Interpretation

## Cluster 1: "Moderately Engaged Users"
- **Recency**: Moderate (52.8 days) → These customers have been somewhat active recently.
- **Frequency (Broadcast)**: Low (102.9 campaigns) → They participate in a modest number of broadcasts.
- **Frequency (Charged)**: Low (78,509 messages) → They send a relatively small number of charged messages.
- **Monetary**: Low (43M) → They generate a modest amount of revenue.

### Characteristics:
- These users are moderately engaged but have low contribution in terms of frequency and revenue.
- They may require targeted campaigns or incentives to increase both their frequency and monetary value.

---

## Cluster 2: "Top Performers"
- **Recency**: Very recent (10.5 days) → These customers are highly active and engaged recently.
- **Frequency (Broadcast)**: High (1,191.5 campaigns) → They conduct a significant number of campaigns.
- **Frequency (Charged)**: High (4.59M messages) → They send a very high number of charged messages.
- **Monetary**: Very high (3.57B) → They generate the highest revenue.

### Characteristics:
- These are your most valuable customers, contributing significantly to both frequency and monetary value.
- Retaining these customers should be a priority through loyalty programs, premium services, or personalized offers.

---

## Cluster 3: "High-Value Frequent Users"
- **Recency**: Recent (29.7 days) → These customers have been active relatively recently.
- **Frequency (Broadcast)**: Very high (2,078.3 campaigns) → They participate in the highest number of campaigns.
- **Frequency (Charged)**: High (422,595 messages) → They send a large number of charged messages.
- **Monetary**: High (152M) → They generate significant revenue.

### Characteristics:
- These users are highly engaged and contribute significantly to frequency and revenue, though not at the level of "Top Performers."
- They represent a valuable segment that should be nurtured and potentially moved toward the "Top Performers" cluster.

---

## Cluster 4: "Dormant or Low-Value Users"
- **Recency**: Very high (262.6 days) → These customers have been inactive for a long time.
- **Frequency (Broadcast)**: Very low (7.9 campaigns) → They conduct very few broadcasts.
- **Frequency (Charged)**: Very low (2,508 messages) → They send minimal charged messages.
- **Monetary**: Very low (1.7M) → They generate negligible revenue.

### Characteristics:
- These customers are disengaged and contribute very little to the business.
- They may require reactivation campaigns or special offers to re-engage them, but some may represent churned users.

---

# Summary of Clusters

| **Cluster** | **Name**                     | **Key Traits**                                                                 |
|-------------|------------------------------|--------------------------------------------------------------------------------|
| **Cluster 1** | "Moderately Engaged Users"   | Modest campaigns, moderate recency, low frequency, and revenue.                |
| **Cluster 2** | "Top Performers"              | Very recent, high campaigns, highest charged messages, and highest revenue.    |
| **Cluster 3** | "High-Value Frequent Users"  | Recent, highest campaigns, high charged messages, significant revenue.         |
| **Cluster 4** | "Dormant or Low-Value Users" | Very inactive, few campaigns, minimal charged messages, negligible revenue.    |

---

# Recommendations
1. **Cluster 1**:
   - Target these users with campaigns to increase their frequency and revenue. 
   - Provide incentives for running more campaigns or expanding their customer engagement.

2. **Cluster 2**:
   - Focus on retention strategies such as loyalty programs, premium services, and personalized experiences to maintain engagement.

3. **Cluster 3**:
   - Nurture this segment by encouraging further participation in campaigns and potentially upselling additional services or packages.

4. **Cluster 4**:
   - Design reactivation campaigns, offer discounts, or provide tailored communication to bring these dormant users back into the active user base.

