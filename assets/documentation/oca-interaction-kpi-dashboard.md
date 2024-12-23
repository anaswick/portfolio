# KPI and Metrics Tracking Dashboard for CRM Services

### Tools used : PostgreSQL, Metabase

In this project, I developed a comprehensive dashboard for tracking key performance indicators (KPIs) and metrics specific to CRM (Customer Relationship Management) services. The dashboard aggregates and visualizes data from multiple customer touchpoints, enabling businesses to monitor user engagement, conversion rates, and transaction activities effectively. Key features include real-time metrics on customer acquisition, activation, retention, and revenue generation. This project demonstrates my expertise in building data-driven solutions for CRM performance analysis, helping stakeholders make informed decisions to optimize customer interactions and drive business growth.

## Dashboard Filters

![image](https://github.com/user-attachments/assets/6a08c020-723b-41c3-8428-748be257c1a7)

**Purpose** : These filters allow users to customize and refine the data displayed in the dashboard, focusing on specific users and time periods. The "User Name" filter enables filtering by individual users, while the "Date Range" filter restricts data to a particular timeframe.

**Usage** : <br>
**User Name**: This filter provides a dropdown list of user names, allowing users to view data and metrics associated with specific individuals. It is particularly useful for analyzing individual user behaviors or performance. 

**Date Range**: This filter enables the selection of a start and end date, allowing users to focus on data within a defined period. It’s essential for tracking changes over time, such as weekly, monthly, or yearly trends.

**Impact** : By applying these filters, users can conduct more targeted analyses, gain insights into individual user activities, and observe trends within specified time periods. This flexibility enhances decision-making by enabling stakeholders to examine specific cases or trends, ultimately leading to a more personalized and time-sensitive understanding of the data.

## Total Current Licensed User by User Type

![image](https://github.com/user-attachments/assets/815198e7-d401-47bf-ae67-2e409f69806a)

**Purpose** <br>
This section provides a count of active users segmented by user type, including all users, admins, owners, supervisors, non-agents, and agents. It offers an overview of the distribution of different user roles within the organization, helping to understand the composition of the user base.

**Usage** <br>
Each tile represents a specific user type with the total number of users in that category:

**Total Users - All**: Total count of all active users, providing a complete view of user presence. <br>
**Total Users - Admin**: Number of users with administrator roles. <br>
**Total Users - Owner**: Number of users who are designated as owners. <br>
**Total Users - Supervisor**: Number of users assigned a supervisory role. <br>
**Total Users - Non-Agent**: Count of users who do not function as agents. <br>
**Total Users - Agents**: Number of users functioning as agents, who are often the primary operators on the platform. <br>

This breakdown helps in assessing role-based user distribution, allowing stakeholders to monitor and manage the user base effectively.

**Impact** <br>
Understanding the distribution of users by type enables the organization to make informed decisions regarding resource allocation and role-specific planning. It can help identify any gaps in user roles, facilitating better workforce management and operational efficiency. Additionally, this insight supports strategic planning by revealing the concentration of user roles, aiding in targeted training, support, and development efforts for each user category.

## Active License Growth 

![image](https://github.com/user-attachments/assets/cf1fbfe4-20c5-47ec-a9d3-1cc858ee4fdd)

**Purpose** <br>
This chart tracks the growth in the number of active users categorized as agents over time. It shows the cumulative increase in active agent users, allowing stakeholders to monitor the adoption and expansion of agent roles within the platform.

**Usage** <br>
The chart plots the daily total of active agents, displaying trends from January 1, 2023, to the present. The cumulative count rises in a step-like pattern, indicating periods of significant user growth. This visualization helps in identifying when spikes in new active agents occurred, which can be correlated with marketing campaigns, product enhancements, or business expansions.

**Impact** <br>
The growth in active agent users reflects the platform’s success in expanding its user base and gaining market share. Steady growth suggests sustained user interest, while any sudden increases could indicate successful engagement initiatives. By analyzing these growth patterns, the organization can better understand user adoption trends, plan for resource scaling, and make data-driven decisions on future growth strategies. This insight also aids in evaluating the effectiveness of recruitment and onboarding processes for agent users.

**Datamart Query**
```
truncate table mart.interaction_num_agents_agg_v2;

insert into mart.interaction_num_agents_agg_v2
with date_series as (
    select generate_series('2023-01-01'::date, now(), '1 day') as daily_date
)
select
    cast(d.daily_date as date) as "date",
    coalesce(na.full_name, na.application) as "full_name",
    coalesce(sum(na.total_agents), 0) as "total_agents"
from date_series d
left join mart.interaction_num_agents na
    on d.daily_date between na.billing_date and (na.next_billing_date - interval '1 day')
group by
    d.daily_date, 2
order by 
    1 asc
;
```

**Metabase Query**
```
select
    "date", sum("total_agents") as "total agents"
from mart.interaction_num_agents_agg_v2
where {{date_range}}
    and {{user_name}}
group by 1
;
```

## Daily, Weekly, Monthly Active User Trend

![image](https://github.com/user-attachments/assets/edfa8a42-5081-442e-8184-028c7b1bdffd)

**Purpose** <br>
This chart visualizes user engagement by tracking daily active users (DAU), weekly active users (WAU), and monthly active users (MAU) over time. It provides an overview of user activity levels and engagement patterns across different timeframes, helping to understand short-term and long-term user retention.

**Usage** <br>

**DAU (Daily Active Users):** The blue line represents the daily active users, showing day-to-day fluctuations in engagement. Peaks and troughs indicate higher activity on specific days, likely aligning with working days and lower activity on weekends or holidays. <br>
**WAU (Weekly Active Users):** The red line shows weekly active users, providing a broader view of user engagement within each week. <br>
**MAU (Monthly Active Users):** The green line represents monthly active users, offering insight into overall engagement across each month. <br>
This trend chart highlights both immediate engagement (DAU) and sustained usage over time (WAU and MAU), enabling stakeholders to monitor user behavior effectively. <br>

**Impact** <br>
By analyzing DAU, WAU, and MAU trends, the organization can gain insights into user engagement consistency and retention. Stable or increasing MAU and WAU levels indicate good long-term retention, while DAU patterns reveal peak engagement periods. This information is crucial for assessing the health of the user base, identifying opportunities to improve daily engagement, and guiding strategies to increase user retention. It also allows for targeted initiatives to boost user activity during periods of lower engagement, enhancing overall platform usage and satisfaction.

**Datamart Query**
```
truncate mart.interaction_active_users;
    
INSERT INTO mart.interaction_active_users (agent_id, created_at, application, full_name)
SELECT
    cah.agent AS agent_id,
    cah.created_at AT TIME ZONE 'utc' AS created_at,
    cah.application,
    COALESCE(u.full_name, cah.application) AS full_name
FROM
    raw.chat_agent_histories cah
left join raw.applications a on cah.application = a."_id" 
left join raw.users u on a."user" = u."_id" 
WHERE
    cah.created_at BETWEEN '2024-01-01' AND NOW();
```

**Metabase Query**
```
with distinct_users as (
	select
		cast(created_at as date) as "date",
		agent_id
	from mart.interaction_active_users
	where {{date_range}} and {{full_name}}
	group by 1,2
)
select
	a."date",
	(
	    select count(distinct agent_id) 
	    from distinct_users b 
	    where b."date" = a."date"
	) as "dau",
	(
	    select count(distinct agent_id) 
	    from distinct_users b 
	    where b."date" between a."date" - interval '6 days' and a."date" + interval '1 days'
	) as "wau",
	(
	    select count(distinct agent_id) 
	    from distinct_users b 
	    where b."date" between a."date" - interval '27 days' and a."date" + interval '1 days'
	) as "mau"
from distinct_users a
group by 1
order by 1
;
```
## User Proportion by Type

![image](https://github.com/user-attachments/assets/0b572df1-b046-432b-909e-c51518821015)

**Purpose**  <br>
This donut chart illustrates the distribution of users by type, segmented into "regular" and "lite" users. It provides a clear overview of the relative proportions of each user type within the total user base, helping to understand how the user base is divided among different service levels.

**Usage** <br>
**Regular Users:** Represented by the yellow section, regular users make up 74.3% of the total user base. <br>
**Lite Users:** Represented by the teal section, lite users account for 25.7% of the total user base. <br>
**Total Users:** The center of the chart shows the total count of users, which is 17,059 in this example. <br>

This segmentation helps stakeholders quickly assess the balance between regular and lite users, indicating the distribution of users based on service level or product usage.

**Impact** <br>
Understanding the proportion of regular versus lite users aids in resource allocation and marketing strategies. A higher proportion of regular users suggests strong adoption of the full-service offering, while a substantial lite user base may highlight an opportunity for upselling to regular plans. This information supports strategic planning for user retention, product development, and targeted marketing efforts, enabling the organization to align offerings with user preferences and maximize revenue potential.

**Metabase Query**
```
SELECT 
    CASE 
        WHEN ca.is_new IS TRUE THEN 'lite' 
        ELSE 'regular'  
    END AS category, 
    COUNT(*) AS label_count,  -- Count the number of occurrences for each label
    ROUND(COUNT(*)::DECIMAL / SUM(COUNT(*)) OVER () * 100, 2) AS proportion  -- Calculate proportion in percentage
FROM 
    raw.chat_agents ca
LEFT JOIN 
    raw.billingmasters b ON ca.application = b.application 
WHERE 
    b.next_billing_date > NOW() and ca.application not in (select application from raw.master_testing_demo_account mtda)
GROUP BY 
    category
ORDER BY 
    proportion DESC;
```
## Total Conversations by Channel

![image](https://github.com/user-attachments/assets/18aa0004-01d4-41aa-979c-f78bfb5c010a)

**Purpose** <br>
This bar chart displays the total volume of conversations (both inbound and outbound) across different communication channels. It provides an overview of channel usage, showing where the majority of customer interactions are taking place. 

**Usage** <br>
Each bar represents a specific communication channel, with the length of the bar indicating the volume of conversations. Key insights include: <br>

**WhatsApp** : The dominant channel with 16.6 million conversations, indicating it is the primary method for customer interaction. <br>
**Email** : The second most-used channel with 1.5 million conversations. <br>
**Other channels**, such as live chat, Instagram DM, and Telegram, have comparatively lower volumes, ranging from 278.3k for live chat to 703 for webchat.
This chart allows stakeholders to quickly identify the most popular channels and compare conversation volumes across all available options.

**Impact** <br>
Understanding conversation volume by channel helps in resource allocation, ensuring that customer service teams are appropriately staffed and supported on the most active platforms. It also enables the organization to optimize user experience by focusing on high-traffic channels like WhatsApp and email. Additionally, insights from this data can guide future marketing and engagement strategies, focusing efforts on the platforms where users are most active. This analysis ultimately aids in improving customer satisfaction and operational efficiency by aligning resources with demand across communication channels.

**Datamart Query**
```
truncate table mart.interaction_conversations;

-- inserting data with created_at from 2024-01 to 2024-04
insert into mart.interaction_conversations
with user_application as (
	select a._id as application_id, u.full_name
	from (select distinct _id, "user" from raw.applications) as a
	join raw.users as u
		on a."user" = u._id
	where a._id not in (select application from raw.master_testing_demo_account)
)
select
	cast((c.created_at at time zone 'utc' at time zone 'Asia/Jakarta') as date)	as "date",
	ua.application_id, 
    coalesce(ua.full_name, ua.application_id) as full_name, 
    r.channel,
	count(distinct c._id) filter (where c.direction = 'inbound') as total_inbound,
	count(distinct c._id) filter (where c.direction = 'outbound') as total_outbound,
	sum(case when r.status = 'unhandled' then 1 else 0 end) status_unhandled,
	sum(case when r.status = 'handled' then 1 else 0 end) status_handled,
	sum(case when r.status = 'resolved' then 1 else 0 end) status_resolved
from raw.chat_conversations as c
left join raw.chat_rooms as r
	on c.room = r._id
left join user_application as ua
	on r.application = ua.application_id
where (c.created_at at time zone 'utc' at time zone 'Asia/Jakarta') between '2024-01-01 00:00:00.001' and '2024-04-30 23:59:59.000'
	and c.direction in ('inbound','outbound')
	and r.channel is not null
group by 1,2,3,4
order by 1 asc, 2 asc
;

-- inserting data with created_at from 2024-05 to 2024-08
insert into mart.interaction_conversations
with user_application as (
	select a._id as application_id, u.full_name
	from (select distinct _id, "user" from raw.applications) as a
	join raw.users as u
		on a."user" = u._id
	where a._id not in (select application from raw.master_testing_demo_account)
)
select
	cast((c.created_at at time zone 'utc' at time zone 'Asia/Jakarta') as date)	as "date",
	ua.application_id, 
    coalesce(ua.full_name, ua.application_id) as full_name, 
    r.channel,
	count(distinct c._id) filter (where c.direction = 'inbound') as total_inbound,
	count(distinct c._id) filter (where c.direction = 'outbound') as total_outbound,
	sum(case when r.status = 'unhandled' then 1 else 0 end) status_unhandled,
	sum(case when r.status = 'handled' then 1 else 0 end) status_handled,
	sum(case when r.status = 'resolved' then 1 else 0 end) status_resolved
from raw.chat_conversations as c
left join raw.chat_rooms as r
	on c.room = r._id
left join user_application as ua
	on r.application = ua.application_id
where (c.created_at at time zone 'utc' at time zone 'Asia/Jakarta') between '2024-05-01 00:00:00.001' and '2024-08-31 23:59:59.000'
	and c.direction in ('inbound','outbound')
	and r.channel is not null
group by 1,2,3,4
order by 1 asc, 2 asc
;

-- inserting data with created_at from 2024-09 to 2024-12
insert into mart.interaction_conversations
with user_application as (
	select a._id as application_id, u.full_name
	from (select distinct _id, "user" from raw.applications) as a
	join raw.users as u
		on a."user" = u._id
	where a._id not in (select application from raw.master_testing_demo_account)
)
select
	cast((c.created_at at time zone 'utc' at time zone 'Asia/Jakarta') as date)	as "date",
	ua.application_id, 
    coalesce(ua.full_name, ua.application_id) as full_name, 
    r.channel,
	count(distinct c._id) filter (where c.direction = 'inbound') as total_inbound,
	count(distinct c._id) filter (where c.direction = 'outbound') as total_outbound,
	sum(case when r.status = 'unhandled' then 1 else 0 end) status_unhandled,
	sum(case when r.status = 'handled' then 1 else 0 end) status_handled,
	sum(case when r.status = 'resolved' then 1 else 0 end) status_resolved
from raw.chat_conversations as c
left join raw.chat_rooms as r
	on c.room = r._id
left join user_application as ua
	on r.application = ua.application_id
where (c.created_at at time zone 'utc' at time zone 'Asia/Jakarta') between '2024-09-01 00:00:00.001' and '2024-12-31 23:59:59.000'
	and c.direction in ('inbound','outbound')
	and r.channel is not null
group by 1,2,3,4
order by 1 asc, 2 asc
;
```

**Metabase Query**
```
select
    channel,
    sum(total_inbound)+sum(total_outbound) as "inbound + outbound"
from mart.interaction_conversations
where {{date_range}}
    and {{user_name}}
group by 1
order by 2 desc
;
```
## Daily Total Tickets

![image](https://github.com/user-attachments/assets/5a806f60-30c3-43fd-9452-159fb71dcd6e)

**Purpose** <br>
This line chart tracks the daily volume of support or service tickets over time. It provides insights into the demand for support services and highlights patterns in ticket submissions, enabling stakeholders to monitor fluctuations in ticket volume.

**Usage** <br>
The x-axis represents the date, while the y-axis shows the total number of tickets submitted each day. Notable peaks, such as the significant spike in June 2024, indicate periods of unusually high ticket volume, possibly due to a specific event, promotion, or system issue. Regular lower levels of daily tickets reflect typical support demand, which shows some variability but generally maintains a consistent trend.

This visualization helps in quickly identifying both daily patterns and any unusual surges in ticket volume, providing valuable context for operational planning.

**Impact** <br>
Understanding daily ticket volume trends allows the organization to optimize staffing and resource allocation, ensuring adequate support coverage during peak periods. The identification of specific high-activity days can prompt further investigation into root causes, such as feature releases, outages, or seasonal spikes, enabling the team to proactively manage similar events in the future. Ultimately, this data supports effective support operations management and enhances the customer experience by ensuring timely responses during high-demand periods.

**Datamart Query**
```
truncate table mart.interaction_tickets;

insert into mart.interaction_tickets
with user_application as (
	select a._id as application_id, u.full_name
	from (select distinct _id, "user" from raw.applications) as a
	join raw.users as u
		on a."user" = u._id
	where a._id not in (select application from raw.master_testing_demo_account)
		and u.full_name is not null
)
select
	"date", full_name, channel, priority, status,
	count(_id) as total_tickets
from (
	select -- jumlah tiket berdasarkan waktu ter-create
		cast((t.created_at at time zone 'utc' at time zone 'Asia/Jakarta') as date) as "date",
		t._id, ua.full_name,
		t.channel, t.priority, t.status,
		row_number() over (partition by t._id order by t._ingest_ts desc) as row_num
	from raw.tickets as t
	join user_application as ua
		on t.application = ua.application_id
	where t.created_at between '2024-01-01' and '2024-12-31'
) as t
where row_num = 1
group by 1,2,3,4,5
order by 1 asc, 2 asc, 6 desc
;
```

**Metabase Query**
```
select
    "date", sum(total_tickets) as "Total tickets"
from mart.interaction_tickets
where {{date_range}}
    and {{user_name}}
group by 1
order by 1 asc
;
```

## Daily Total Conversations

![image](https://github.com/user-attachments/assets/fd8c12c2-9113-49eb-a698-995678a7d351)

**Purpose** <br>
This line chart tracks the total daily volume of conversations (both inbound and outbound) across various communication channels. It helps monitor daily interaction levels, providing insights into customer engagement trends and support demand over time.

**Usage** <br>
The x-axis represents dates throughout 2024, while the y-axis shows the number of conversations per day. Regular fluctuations reflect typical engagement patterns, while significant spikes, such as those around June and October, indicate periods of increased conversation activity. These peaks may be associated with events, promotions, or technical issues that required additional customer interaction.

This visualization allows for quick identification of both routine patterns and unusual surges in conversation volume, offering a basis for operational adjustments.

**Impact** <br>
Analyzing daily conversation volume enables the organization to optimize resource allocation and ensure adequate support staffing during high-demand periods. Recognizing peak days allows for proactive planning, which helps maintain service quality during busy times. This data also informs future strategies by identifying patterns in customer engagement, allowing the organization to plan targeted initiatives that enhance customer experience and efficiently handle fluctuations in demand.

**Metabase Query**
```
select
    "date",
    sum(total_inbound)+sum(total_outbound) as "inbound + outbound"
from mart.interaction_conversations
where {{date_range}}
    and {{user_name}}
group by 1
order by 1 asc
;
```
