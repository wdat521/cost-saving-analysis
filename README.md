# Cost Saving Analysis
A cost analysis of  telecommunications inventory (cables, DIA, etc.) focusing on cutting unnecessary costs.


# <img width="612" height="612" alt="image" src="https://github.com/user-attachments/assets/700e1f67-f7aa-43d8-80ef-1158ee83e995" /> Problem Statement 
The company is spending roughly 30% of its annual budget for inventory networking services and equipment management. Due to the 3-5% price increase of our suppliers, we want to strategically reduce the inventory budget to 10%, factoring in capital, storage, insurance, and obsolescence. By the end of 2026, we want to save at least 1M USD.

Below are strategies that could generate cost savings:
  * Identify circuits that were requested for termination but are still being billed (services that are inactive, decommissioned, but are still being billed). 
  * Identify circuits that are potentially duplicate routes (the same product type, and A and Z locations).
  * Identify circuits that are underused (utilization percentage < 20%).
  * Identify out of term contracts but are still being billed to reassess usability.

# <img width="1200" height="1200" alt="image" src="https://github.com/user-attachments/assets/9cf03a23-22f1-45bc-9ec7-885efa00bac2" /> Dataset
  * circuit_id - service reference of circuits
  * monthly_recurring_cost - unit price
  * a_end - A location
  * z_end - Z location
  * product_type - whether it's a Cross Connect, Dark Fiber, Fiber, Internet DIA, Metro Fiber, Wave
  * supplier
  * start_date - contract start date
  * end_date - contract end date
  * contract_term_months - contract length in months
  * billing_status - whether the circuits are still being billed or not
  * decom_status - decommission status of circuits
  * service_status - 
  * reclaim - eligibility for reclaim or credit note
  * reclaim_total - total reclaim out of the cost saving project
  * utilization_pct - utilized percentage of the circuit capacity

# <img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/5e73b302-e555-445d-a7a7-509bdbf41b1f" /> Methodology
## Tools Used:
* Python for data cleaning, data imputation, and exploratory data analysis
* Power BI for visualization

## Exploring Dataframe Dimension
The dataframe 'cs' has 2120 rows and 15 columns. It is also notable that the column end_date has empty values. Below is also the first five rows of the dataframe.
  
<img width="460" height="577" alt="image" src="https://github.com/user-attachments/assets/75b566af-faf0-4d69-a9cd-36907489e2b4" />

|index|circuit\_id|monthly\_recurring\_cost|a\_end|z\_end|product\_type|supplier|start\_date|end\_date|contract\_term\_months|billing\_status|decom\_status|service\_status|reclaim|reclaim\_total|utilization\_pct|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|CKT-07237|2824|Dallas DC1|London DC2|Internet DIA|Orange|13-11-22|12-11-27|36|BILLING|PENDING DECOM|Provisioning|NaN|NaN|34|
|1|CKT-07085|350|Frankfurt DC1|Zurich DC1|Fiber|BT|2020-01-18|16-01-25|24|Not Billed|DECOM|Provisioning|NaN|NaN|75|
|2|CKT-02822|1316|London DC1|Tokyo DC1|Wave|Orange|2021/11/16|2024-11-15|36|billing|DECOM|Active|NaN|NaN|55|
|3|CKT-07997|4564|Zurich DC1|Amsterdam DC1|Metro Fiber|BT|28-01-21|NaN|36|BILLING|ACTIVE|Pending Disconnect|NaN|NaN|94|
|4|CKT-03339|150|Tokyo DC1|London DC1|Cross Connect|Lumen|2021/09/24|23-09-26|60|BILLING|PENDING DECOM|Provisioning|NaN|NaN|61|

## Format Checking and Standardization
- No empty values in columns circuit_id, a_end, z_end, product_type, supplier, start_date, monthly_recurring_cost, contract_termn_months, decom_status, and utilization_pct. Values are also in the smae format. Note that circuit_id doesn't have to be unique.

- A reclaim or credit note can be expected if the circuit was reqeusted for termination, the vendor acknowledged request, yet we are still being billed. Hence looking at the dataset, the circuit has to be in billing state, decommissioned, and the service it is used for is inactive. Upon checking, the columns reclaim and reclaim_total are logical.

<img width="1385" height="539" alt="Screenshot 2026-03-18 025726" src="https://github.com/user-attachments/assets/a8337c5c-b43d-48a0-9dcf-041e467d7e3a" />

- The categories for column billing_status is shown as: 'ACTIVE BILLING', 'BILLING', 'Not Billed', 'billing'. Categories 'ACTIVE BILLING', 'BILLING', and , 'billing' are the same in meaning.

<img width="445" height="86" alt="image" src="https://github.com/user-attachments/assets/d69ad234-7ede-4e8e-bb0a-c79c16c87194" />

- Standardizing the billing_status column, a new column called clean_billing_status is appended to the dataframe. Column billing_status can be removed.

<img width="343" height="235" alt="image" src="https://github.com/user-attachments/assets/2b18db38-609b-4dad-9061-69ea37dfba6e" />
<img width="1380" height="372" alt="image" src="https://github.com/user-attachments/assets/d770a8fb-bbec-44db-8518-9aaf77c859c8" />

- Checking the service_status column, the categories 'Active' and 'active' are the same. To fix this:

<img width="575" height="107" alt="image" src="https://github.com/user-attachments/assets/edf06665-75a8-453c-8c8c-2567175bdaff" />
  
- It is notable that the columns start_date and end_date are not uniform in format. 

<img width="823" height="355" alt="image" src="https://github.com/user-attachments/assets/aaf8b3ed-6ee4-4964-b486-a1b902b2d634" />
<img width="1400" height="481" alt="image" src="https://github.com/user-attachments/assets/6b13b22b-0680-4c22-b7d5-bdcbe3a24c89" />

Filling in empty end_date by adding the contract terms in monrth to the start date.

<img width="822" height="490" alt="image" src="https://github.com/user-attachments/assets/26322b21-4279-4c4d-b6d3-5a4e9d6188e1" />

# Analyses

## Cost-saving Objective I: Identify circuits that were requested for termination but are still being billed (services that are inactive, decommissioned, but are still being billed).
Flagging rows that are cost savings (decom_status = DECOM, billing_status = BILLING, service_status = INACTIVE)

<img width="1396" height="726" alt="image" src="https://github.com/user-attachments/assets/9de8228b-4d9c-4499-a88d-12f93171a620" />

## Cost-saving Objective II: identify circuits that are potentially duplicate routes (the same product type, and A and Z locations).
decom_status = ACTIVE, clean_billing_status = BILLING, service_status = ACTIVE and PROVISIONING)

<img width="1393" height="664" alt="image" src="https://github.com/user-attachments/assets/b6cb0c8f-0d3b-4966-9005-019a9ad74465" />

## Cost-saving Objective III: identify circuits that are underused (utilization percentage > 20%).
Flagging underused circuits with less than 20% utilization rate

<img width="1379" height="439" alt="image" src="https://github.com/user-attachments/assets/c788e00a-431d-4fe9-ae47-11b2b932a04c" />

## Cost-saving Objective IV: identify out of term contracts but are still being billed to reassess usability.

<img width="859" height="472" alt="image" src="https://github.com/user-attachments/assets/2b8717ef-813d-4028-9a64-0afd305cc73f" />

Upon further look, inconsistencies are detected in the end_date and contract_term_months columns.
As an example, circuit CKT-07237 start date was on 2022-11-13, with a 36 contract term in months. Its contract should have ended on 2025-11-13, but it shows 2027-12-11 instead.
Further investigation on contract terms is highly recommended.
