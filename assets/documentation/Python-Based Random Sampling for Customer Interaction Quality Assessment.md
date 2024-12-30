# Python-Based Random Sampling for Customer Interaction Quality Assessment

### Tools used : Google Big Query, Python, Excel

**Description** <br>
The goal of this project is to improve customer service quality and ensure interactions meet established standards by implementing a Python-based random sampling and scoring system for customer service interactions. This system adheres to the principle of continuous improvement, enabling regular assessments and refinements to service quality. The sampling focuses specifically on complaint tickets, ensuring that the most critical customer interactions are evaluated for quality and resolution effectiveness.

This Python notebook automates the random sampling and scoring process by: <br>

**Random Sampling of Complaint Tickets**: Periodically selecting a defined number of complaint tickets per agent, ensuring that a representative sample of the most impactful interactions is assessed. <br>
**Assigning Quality Assessors**: Distributing tickets evenly among assessors using a round-robin method with added randomization to prevent repeated assignments to the same assessors.
**Scoring Framework**: Allowing assessors to review and score interactions based on predefined criteria, such as clarity, professionalism, empathy, and resolution effectiveness.

**Outcomes**<br>
The project delivers a scalable, data-driven solution for ongoing quality assurance in customer service. Key benefits include:

- Continuous Improvement: Regular assessments ensure that customer service standards are constantly refined, leading to enhanced interaction quality over time.
- Targeted Evaluation: By focusing on complaint tickets, the system prioritizes the evaluation of interactions that matter most to customers and the organization.
- Agent Development: Insights from the scoring process help identify training needs and recognize top-performing agents, fostering a culture of excellence.

This system enables the organization to uphold high service standards, improve customer satisfaction, and build loyalty by addressing and resolving customer complaints effectively. It showcases how Python can be utilized to drive quality assurance and continuous improvement in a customer service environment.

## Project Step-by-step


**Import Library**

```
import pandas as pd
```

**Load the datasets**
```
agents_df = pd.read_excel('agent_name.xlsx')  # File containing the list of current customer care agents
tickets_df = pd.read_csv('202411_w1.csv')  # File containing ticket data
mapping_df = pd.read_excel('mapping_agent_doa.xlsx', sheet_name='Sheet1')  # File with agent name, assignee_id, and reporter_id

# Extract the list of agent names to be checked
agents_to_check = agents_df['agent_name'].tolist()

# Merge to get agent_name_assignee by joining tickets_df with mapping_df on assignee_id
tickets_with_assignee = tickets_df.merge(mapping_df[['assignee_id', 'agent_name']], 
                                         on='assignee_id', 
                                         how='left').rename(columns={'agent_name': 'agent_name_assignee'})

# Merge to get agent_name_reporter by joining tickets_with_assignee with mapping_df on reporter_id
tickets_with_agent_names = tickets_with_assignee.merge(mapping_df[['reporter_id', 'agent_name']], 
                                                       on='reporter_id', 
                                                       how='left').rename(columns={'agent_name': 'agent_name_reporter'})

# Filter tickets to include only those agents listed in agent.xlsx (filter by agent_name_assignee)
filtered_tickets_df = tickets_with_agent_names[tickets_with_agent_names['agent_name_assignee'].isin(agents_to_check)]

# Save the filtered result to a new Excel file
output_file = 'tickets_202411w1_agent_filtered.xlsx'  # Filtered tickets based on agent names
filtered_tickets_df.to_excel(output_file, index=False)

print(f"Filtered tickets have been saved to {output_file}")
```

**Filtering the data to make sure there is no tickets from SPAM and INFRATRUCTURE category**

```
# Load the filtered tickets dataset
filtered_tickets_df = pd.read_excel('tickets_202411w1_agent_filtered.xlsx')  # Adjust the file name accordingly

# Ensure the 'category' and 'channel' columns are treated as strings
filtered_tickets_df['category'] = filtered_tickets_df['category'].astype(str)
filtered_tickets_df['channel'] = filtered_tickets_df['channel'].astype(str)

# Filter out tickets from the categories "SPAM", "Spam", "spam"
filtered_tickets_df = filtered_tickets_df[~filtered_tickets_df['category'].str.lower().isin(['spam'])]

# Filter out tickets from the channel "Form", "FORM", "form"
filtered_tickets_df = filtered_tickets_df[~filtered_tickets_df['channel'].str.lower().isin(['form'])]

# Filter out tickets where the category column contains 'inf -%%' pattern
filtered_tickets_df = filtered_tickets_df[~filtered_tickets_df['category'].str.contains(r'inf -%%', case=False, na=False)]

# Filter to include only rows where agent_name_reporter is equal to agent_name_assignee
filtered_tickets_df = filtered_tickets_df[filtered_tickets_df['agent_name_reporter'] == filtered_tickets_df['agent_name_assignee']]

# Save the filtered result to a new Excel file
output_file = 'filtered_tickets_no_spam_no_form_no_inf_matching_agents_202411w1.xlsx'
filtered_tickets_df.to_excel(output_file, index=False)

print(f"Filtered tickets (without SPAM, Form channels, 'inf -%%' pattern, and matching agents) have been saved to {output_file}")
```

**Run the random sampling and check the result of sampling**
```
import pandas as pd
import random

# Load the filtered tickets and assessors datasets
filtered_tickets_df = pd.read_excel('filtered_tickets_no_spam_no_form_no_inf_matching_agents_202411w1.xlsx')
assessors_df = pd.read_excel('assessor.xlsx')

# Extract the list of quality assessors
assessors = assessors_df['assessor'].tolist()
assessors_count = len(assessors)

# Function to sample 3 tickets per agent
def sample_tickets(group):
    # Check if there are at least 3 tickets; if not, sample all available tickets
    if len(group) >= 3:
        return group.sample(n=3, random_state=42)
    else:
        return group  # If fewer than 3 tickets, return all available tickets

# Apply the sampling function to each agent
sampled_tickets_df = filtered_tickets_df.groupby('agent_name_assignee').apply(sample_tickets).reset_index(drop=True)

# Shuffle the assessors list to ensure random distribution each time
random.shuffle(assessors)

# Ensure equal distribution of tickets among assessors using the shuffled list
sampled_tickets_df['QualityAssessor'] = [
    assessors[i % assessors_count] for i in range(len(sampled_tickets_df))
]

# Save the sampled and assigned tickets to a new Excel file
output_file = 'chosen_tickets_3_per_agent_equally_distributed_202411w1.xlsx'
sampled_tickets_df.to_excel(output_file, index=False)

print(f"Chosen tickets have been saved to {output_file}")
```


**The expected result should suffice this conditions**
- Exclude tickets from the spam and infrastructure categories.
- Ensure tickets are equally distributed among agents.
- Each Quality Assessor (Agent Supervisor) must review samples from several different agents.

### Result

![image](https://github.com/user-attachments/assets/254b63e2-0fff-49ce-a72e-0931082a25b4)

