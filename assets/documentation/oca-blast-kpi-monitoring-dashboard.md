# KPI and Metrics Tracking Dashboard for Omnichannel Communication Assistant

### Tools used : PostgreSQL, Metabase

In this project, my team developed an interactive dashboard to monitor key performance indicators (KPIs) and metrics for an omnichannel communication assistant platform. The dashboard aggregates and visualizes data from multiple communication channels, including WhatsApp, email, SMS, and IVR, to provide a comprehensive view of customer interactions and engagement.

Key features include real-time tracking of message delivery success rates, engagement metrics per channel, and usage trends. The dashboard empowers stakeholders to make data-driven decisions, optimize channel performance, and improve customer retention by identifying insights across various communication touchpoints. This project demonstrates my ability to create impactful, data-driven solutions for complex, multi-channel environments.

## Dashboard Filters

![image](https://github.com/user-attachments/assets/451abde5-223f-4028-b6b6-1af9c14f85ec)

The dashboard includes three primary filters to help users refine data views and conduct detailed analysis:

**1. Date Range Filter** <br>

**Purpose**: This filter allows users to select a specific time period to view data, making it easier to analyze trends and patterns over a chosen range. <br>

**Usage**: Users can set a custom start and end date to define the time range they’re interested in. By default, the filter in this example is set from January 1, 2024, to November 15, 2024. <br>

**Impact**: Adjusting the date range provides flexibility in examining data over monthly, quarterly, or yearly spans or any custom period, which is particularly useful for tracking seasonal trends and year-over-year changes. <br>

**2. Channel Filter** <br>

**Purpose**: This filter lets users narrow down the data by specific communication channels (e.g., WhatsApp, email, SMS, IVR). <br>

**Usage**: Selecting a channel filters the data to display only interactions and metrics related to the chosen communication platform. <br>

**Impact**: By isolating a single channel, users can analyze the performance and effectiveness of that channel, comparing engagement metrics and KPIs across different communication platforms to understand which channels are driving the most engagement. <br>

**3. Category Type Filter** <br>

**Purpose**: This filter allows users to view data by specific categories or types within the dataset. Categories may include different types of messages or interactions (e.g., promotional, transactional). <br>

**Usage**: Users can select one or more categories to focus on specific data types, making it easier to distinguish between various types of interactions. <br>

**Impact**: Filtering by category type provides insights into the performance and behavior of different categories, such as evaluating the effectiveness of promotional vs. transactional messages. This level of granularity supports targeted analysis and decision-making. <br>

## User Engagement and Activity Summary

![image](https://github.com/user-attachments/assets/b2ad4f6b-6c65-4181-bd13-6189952576b6)

**1. Total Registered Users**

**Purpose**: To measure the total number of users who have signed up for the platform, reflecting user acquisition and overall reach. <br>

**Usage**: This metric provides insight into the platform’s growth over time. It is used to assess the effectiveness of marketing campaigns, onboarding processes, and user acquisition strategies. <br>

**Impact**: A steady increase in registered users indicates successful outreach and brand awareness, contributing to a larger user base and potential for increased engagement and revenue. <br>


**2. Active Users**

**Purpose**: To track the number of users actively engaging with the platform within a specified period, indicating retention and ongoing interest. <br>

**Usage**: This metric is used to evaluate user engagement and satisfaction by monitoring how many users return to the platform regularly. It can help assess the effectiveness of features designed to encourage continued usage. <br>

**Impact**: A high number of active users suggests the platform is meeting user needs, enhancing retention, and sustaining user interest. This leads to a healthier user base, as engaged users are more likely to convert into transacting users and loyal customers. <br>

**3. Total Transacting Users**

**Purpose**: To measure the number of users completing transactions (e.g., purchases, message blasts) within a given period, which directly relates to revenue generation. <br>

**Usage**: This metric highlights the success of the platform in converting active users into paying users. It is used to evaluate monetization efforts and understand user behavior related to transactions. <br>

**Impact**: An increase in transacting users reflects higher revenue potential and successful conversion of active users to paying customers. This metric provides critical insights into monetization and helps identify opportunities to improve conversion rates and optimize revenue streams. <br>

![image](https://github.com/user-attachments/assets/5a4e20af-6b90-48bc-9e84-9e4b7ce16432)

This section of the dashboard visualizes the trends in New Registered Users and Transacting Users over time.

**New Registered Users Trend**: This line chart displays the number of new users registering on the platform over the course of the year. By tracking registration spikes and dips, stakeholders can identify periods of high user acquisition, possibly aligning with marketing efforts or seasonal demand.

**Transacting Users Trend**: This line chart shows the number of users completing transactions over time. It provides insights into revenue-generating activity, allowing the team to analyze patterns in user transactions and assess the effectiveness of strategies to drive user engagement and spending.

Together, these charts offer a clear view of user growth and engagement trends, enabling data-driven decisions to optimize acquisition and conversion strategies.

## Active Users Trends

![image](https://github.com/user-attachments/assets/74aaf236-9531-4eea-a268-26292de9ca4b)

**Purpose** : This chart visualizes the trends in Daily Active Users (DAU), Weekly Active Users (WAU), and Monthly Active Users (MAU) over time. It provides a clear view of user engagement levels across different time frames, showing how often users are interacting with the platform on a daily, weekly, and monthly basis. <br>

**Usage** : The DAU line represents the number of unique users engaging with the platform each day, with peaks typically aligning with working days. WAU and MAU lines show the count of users active within a weekly and monthly period, respectively. This breakdown helps identify short-term and long-term engagement trends, with the DAU line reflecting day-to-day activity and WAU and MAU providing a broader view of ongoing user retention. <br>

**Impact** : Tracking DAU, WAU, and MAU helps stakeholders understand user behavior and engagement patterns, especially in a context where most usage occurs on working days. Regular fluctuations in DAU, alongside relatively stable WAU and MAU values, indicate consistent usage patterns that align with work schedules. This data enables the team to assess user retention over time and make informed decisions on strategies to boost daily engagement or maintain long-term user loyalty. <br>


## Heatmap

### Busy Period by Number of Active Users

![image](https://github.com/user-attachments/assets/91580bac-0d45-44ec-91fc-c123f75672e4)

**Purpose** : This heatmap visualizes user activity levels across different hours of the day and days of the week for the OCA (Omnichannel Communication Assistant) blast feature. It highlights the busy periods when the highest number of users are active, helping to identify peak times for platform usage. <br>

**Usage** : The rows represent each day of the week, while the columns show each hour of the day (0–23). The color intensity varies, with darker shades indicating lower user counts and brighter shades (yellow to red) representing higher user counts. For example, peak hours are commonly seen between 8 AM and 12 PM on weekdays, particularly on Tuesday and Wednesday. This layout enables easy identification of high-traffic periods throughout the week. <br>

**Impact** : Understanding the busiest hours for OCA blasts allows the team to optimize resource allocation, system performance, and support availability during peak usage times. This data-driven approach ensures that the platform can handle demand efficiently during peak hours, minimizing the risk of downtime and enhancing user experience. It also helps with scheduling maintenance during low-activity periods and planning targeted marketing or support activities to engage users during high-traffic times. <br>

### Busy Period by Number of Charged Message

![image](https://github.com/user-attachments/assets/e903bfc7-a6e9-4a30-ba87-01e08aef478b)

**Purpose** : This heatmap visualizes the distribution of charge transactions for the OCA (Omnichannel Communication Assistant) blast feature across different hours of the day and days of the week. It highlights periods with the highest volume of transactions, indicating peak times when users are actively engaging in chargeable activities. <br>

**Usage** : Each row represents a day of the week, and each column represents an hour of the day (0–23). The color gradient reflects transaction volume, with darker shades indicating lower volumes and brighter shades (yellow to red) representing higher volumes. This chart shows significant peaks on weekdays, especially between 8 AM and 3 PM, with Tuesday and Wednesday being particularly busy. The heatmap allows quick identification of high-transaction periods throughout the week. <br>

**Impact** : Understanding peak transaction times helps optimize platform performance and resource management to handle high transaction volumes effectively. This information allows for strategic planning, such as scaling resources during peak hours to ensure smooth operation and preventing downtime. Additionally, insights from this heatmap can guide promotional and support activities to align with high-activity periods, improving user experience and revenue generation during busy times. <br>

## Total Blast by Channel

![image](https://github.com/user-attachments/assets/0e9f2a5b-e64b-46ac-8995-0cd5bbfaa2db)

**Purpose** : This bar chart displays the total charges associated with message blasts across different communication channels, including WhatsApp, email, IVR, and SMS. The chart provides a comparative view of each channel’s contribution to total revenue from message blasts.

**Usage** : Each bar represents a channel, with the length of the bar indicating the total charge generated by that channel. For example, WhatsApp has the highest total charge at 17.8 million, followed by email at 3.6 million, IVR at 1.3 million, and SMS at 255.8k. This visualization makes it easy to identify which channels are generating the most revenue from message blasting.

**Impact** : Understanding the revenue contribution from each channel allows stakeholders to prioritize resources and investments toward the most profitable channels. For instance, the high charge volume on WhatsApp suggests it is the most effective channel, potentially warranting further development or promotion. Conversely, lower-performing channels like SMS might benefit from optimization or could be phased out if they are not cost-effective. This analysis helps optimize channel performance, improve revenue streams, and shape future strategic decisions.

## Invitation Funneling

![image](https://github.com/user-attachments/assets/94749174-4fcf-439b-836b-fbd20fbe1e3f)

**Purpose** :
This funnel chart visualizes the conversion journey of users through different stages of engagement with the blast invitation. It tracks the progression from users who were invited, to those who activated, and finally to those who completed a transaction, highlighting the drop-off rates at each stage.

**Usage** :
The chart begins with the total number of invited users (210), followed by activated users (50% or 105), and transacting users (16.19% or 34). Each stage narrows, representing a decrease in user count as they progress through the funnel. This visual allows stakeholders to assess the effectiveness of the invitation and activation process, as well as the conversion to transaction.

**Impact** : 
Understanding conversion rates at each stage of the funnel provides insights into areas where users may be disengaging. For example, a high drop-off rate between activated and transacting users might suggest improvements are needed in the transaction experience or incentives. By identifying and addressing these gaps, the team can enhance the overall conversion rate, optimize the user journey, and ultimately increase revenue through improved engagement.
