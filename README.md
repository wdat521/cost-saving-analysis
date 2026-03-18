# 📌 Cost Saving Analysis
A cost analysis of  telecommunications inventory (cables, DIA, etc.) focusing on cutting unnecessary costs.


## 🧩 The Challenge 
The company is spending roughly 30% of its annual budget for inventory networking services and equipment management. Due to the 3-5% price increase of our suppliers, we want to strategically reduce the inventory budget to 10%, factoring in capital, storage, insurance, and obsolescence. By the end of 2026, we want to save at least 1M USD.

Below are strategies that could generate cost savings:
  * Identify circuits that were requested for termination but are still being billed (services that are inactive, decommissioned, but are still being billed). 
  * Identify circuits that are potentially duplicate routes (the same product type, and A and Z locations).
  * Identify circuits that are underused (utilization percentage < 20%).
  * Identify out of term contracts but are still being billed to reassess usability.

## 📊 Dataset
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

## ⚙️ Methodology
Tools Used:
* Python for data cleaning, data imputation, and exploratory data analysis
* Power BI for visualization
  
## DataFrame Overview

**Type:** `pandas.core.frame.DataFrame`

**Range Index:** 2120 entries, from 0 to 2119

## Data Columns (Total 15 columns):

| # | Column | Non-Null Count | Dtype |
|---|---------|----------------|--------|
| 0 | circuit_id | 2120 non-null | object |
| 1 | monthly_recurring_cost | 2120 non-null | int64 |
| 2 | a_end | 2120 non-null | object |
| 3 | z_end | 2120 non-null | object |
| 4 | product_type | 2120 non-null | object |
| 5 | supplier | 2120 non-null | object |
| 6 | start_date | 2120 non-null | object |
| 7 | end_date | 1598 non-null | object |
| 8 | contract_term_months | 2120 non-null | int64 |
| 9 | billing_status | 2120 non-null | object |
|10 | decom_status | 2120 non-null | object |
|11 | service_status | 2120 non-null │ object|
df: float64(1), int64(3), object(11)

First five rows of the dataframe ```cs```:
|index|circuit\_id|monthly\_recurring\_cost|a\_end|z\_end|product\_type|supplier|start\_date|end\_date|contract\_term\_months|billing\_status|decom\_status|service\_status|reclaim|reclaim\_total|utilization\_pct|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|CKT-07237|2824|Dallas DC1|London DC2|Internet DIA|Orange|13-11-22|12-11-27|36|BILLING|PENDING DECOM|Provisioning|NaN|NaN|34|
|1|CKT-07085|350|Frankfurt DC1|Zurich DC1|Fiber|BT|2020-01-18|16-01-25|24|Not Billed|DECOM|Provisioning|NaN|NaN|75|
|2|CKT-02822|1316|London DC1|Tokyo DC1|Wave|Orange|2021/11/16|2024-11-15|36|billing|DECOM|Active|NaN|NaN|55|
|3|CKT-07997|4564|Zurich DC1|Amsterdam DC1|Metro Fiber|BT|28-01-21|NaN|36|BILLING|ACTIVE|Pending Disconnect|NaN|NaN|94|
|4|CKT-03339|150|Tokyo DC1|London DC1|Cross Connect|Lumen|2021/09/24|23-09-26|60|BILLING|PENDING DECOM|Provisioning|NaN|NaN|61|

## Data Preprocessing
- No empty values in columns circuit_id, a_end, z_end, product_type, supplier, start_date, monthly_recurring_cost, contract_termn_months, decom_status, and utilization_pct. Values are also in the smae format. Note that circuit_id doesn't have to be unique.

- The categories for column billing_status is shown as: 'ACTIVE BILLING', 'BILLING', 'Not Billed', 'billing'. Categories 'ACTIVE BILLING', 'BILLING', and , 'billing' are the same in meaning.


`sorted(cs['billing_status'].unique())`


['ACTIVE BILLING', 'BILLING', 'Not Billed', 'billing']


- Standardizing the billing_status column, a new column called clean_billing_status is appended to the dataframe. Column billing_status can be removed.


`standardized = []`

`for c in cs['billing_status']:`
   ` if "billing" in c.lower():`
       ` standardized.append("BILLING")`
   ` else:`
       ` standardized.append("NOT BILLED")`
       
`cs['clean_billing_status'] = standardized`
`cs.head()`

Removing the column billing_status as it is replaced by column clean_billing_status


`cs.drop(columns="billing_status", inplace=True)`
`cs.head()`


|index|circuit\_id|monthly\_recurring\_cost|a\_end|z\_end|product\_type|supplier|start\_date|end\_date|contract\_term\_months|decom\_status|service\_status|reclaim|reclaim\_total|utilization\_pct|clean\_billing\_status|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|CKT-07237|2824|Dallas DC1|London DC2|Internet DIA|Orange|13-11-22|12-11-27|36|PENDING DECOM|Provisioning|NaN|NaN|34|BILLING|
|1|CKT-07085|350|Frankfurt DC1|Zurich DC1|Fiber|BT|2020-01-18|16-01-25|24|DECOM|Provisioning|NaN|NaN|75|NOT BILLED|
|2|CKT-02822|1316|London DC1|Tokyo DC1|Wave|Orange|2021/11/16|2024-11-15|36|DECOM|Active|NaN|NaN|55|BILLING|
|3|CKT-07997|4564|Zurich DC1|Amsterdam DC1|Metro Fiber|BT|28-01-21|NaN|36|ACTIVE|Pending Disconnect|NaN|NaN|94|BILLING|
|4|CKT-03339|150|Tokyo DC1|London DC1|Cross Connect|Lumen|2021/09/24|23-09-26|60|PENDING DECOM|Provisioning|NaN|NaN|61|BILLING|

- A reclaim or credit note can be expected if the circuit was reqeusted for termination, the vendor acknowledged request, yet we are still being billed. Hence looking at the dataset, the circuit has to be in billing state, decommissioned, and the service it is used for is inactive. Upon checking, the columns reclaim and reclaim_total are logical.

`reclaim_columns = ['circuit_id','decom_status', 'service_status','clean_billing_status']`
`reclaim_condition = cs['reclaim'] == 'YES' `

`reclaim_df = cs.loc[reclaim_condition, reclaim_columns]`
`reclaim_df.head()`

|index|circuit\_id|decom\_status|service\_status|clean\_billing\_status|
|---|---|---|---|---|
|44|CKT-00705|DECOM|Inactive|BILLING|
|84|CKT-09044|DECOM|Inactive|BILLING|
|194|CKT-07283|DECOM|Inactive|BILLING|
|314|CKT-08592|DECOM|Inactive|BILLING|
|327|CKT-08965|DECOM|Inactive|BILLING|

- Checking the service_status column, the categories 'Active' and 'active' are the same. To fix this:

`# Format checking: service_status`
`sorted(cs.service_status.unique())`

['Active', 'Inactive', 'Pending Disconnect','Provisioning', 'Suspended', 'active']

`cs['service_status'] = cs.service_status.str.upper()`
`sorted(cs.service_status.unique())`

['ACTIVE', 'INACTIVE', 'PENDING DISCONNECT', 'PROVISIONING', 'SUSPENDED']

- It is notable that the columns start_date and end_date are not uniform in format. 

`cs['start_date'] = pd.to_datetime(cs['start_date'])`
`cs['end_date'] = pd.to_datetime(cs['end_date'])`
`cs.head()`

|index|start\_date|end\_date|
|---|---|---|
|0|2022-11-13 00:00:00|2027-12-11 00:00:00|
|1|2020-01-18 00:00:00|2025-01-16 00:00:00|
|2|2021-11-16 00:00:00|2024-11-15 00:00:00|
|3|2021-01-28 00:00:00|NaT|
|4|2021-09-24 00:00:00|2026-09-23 00:00:00|

- Filling in empty end_date by adding the contract terms in monrth to the start date.

`blank_end_date = cs['end_date'].isna()`

`cs.loc[blank_end_date, 'end_date'] = cs.loc[blank_end_date].apply(`
    `lambda x: x['start_date'] + pd.DateOffset(months=x['contract_term_months']),`
    `axis=1`
`)`

`cs['end_date'].isna()`

| Index | End Date | 
|-------|----------|
| 0     | False    |
| 1     | False    |
| 2     | False    |
| 3     | False    |
| 4     | False    |
| ...   | ...      |
| 2115  | False    |
| 2116  | False    |
| 2117  | False    |
| 2118  | False    |
| 2119  | False    |

**DataFrame Dimensions:**
- Rows: 2120
- Columns: 1
- Data Type: bool

## 🔍 Analyses

### Cost-saving Objective I: Identify circuits that were requested for termination but are still being billed (services that are inactive, decommissioned, but are still being billed).
Flagging rows that are cost savings (decom_status = DECOM, billing_status = BILLING, service_status = INACTIVE)

<img width="1396" height="726" alt="image" src="https://github.com/user-attachments/assets/9de8228b-4d9c-4499-a88d-12f93171a620" />

### Cost-saving Objective II: identify circuits that are potentially duplicate routes (the same product type, and A and Z locations).
decom_status = ACTIVE, clean_billing_status = BILLING, service_status = ACTIVE and PROVISIONING)

<img width="1393" height="664" alt="image" src="https://github.com/user-attachments/assets/b6cb0c8f-0d3b-4966-9005-019a9ad74465" />

### Cost-saving Objective III: identify circuits that are underused (utilization percentage > 20%).
Flagging underused circuits with less than 20% utilization rate

<img width="1379" height="439" alt="image" src="https://github.com/user-attachments/assets/c788e00a-431d-4fe9-ae47-11b2b932a04c" />

### Cost-saving Objective IV: identify out of term contracts but are still being billed to reassess usability.

<img width="859" height="472" alt="image" src="https://github.com/user-attachments/assets/2b8717ef-813d-4028-9a64-0afd305cc73f" />

Upon further look, inconsistencies are detected in the end_date and contract_term_months columns.
As an example, circuit CKT-07237 start date was on 2022-11-13, with a 36 contract term in months. Its contract should have ended on 2025-11-13, but it shows 2027-12-11 instead.
Further investigation on contract terms is highly recommended.

## 💡 Insights

## 📈 Recommendations
