# 📌Network Inventory Cost Saving Analysis
This project focuses on identifying unnecessary networking inventory costs.


## 🧩 The Challenge 
The company is spending roughly 30% of its annual budget for inventory networking services and equipment management. Due to the 3-5% price increase of our suppliers, we want to strategically reduce the inventory budget to 10%, factoring in capital, storage, insurance, and obsolescence. By the end of 2026, we want to save at least 1M USD.

Below are strategies that could generate cost savings:
  * Identify circuits that were requested for termination but are still being billed (customer services that are inactive, decommissioned inventory, yet still being billed by the vendor). 
  * Identify circuits that are potentially duplicate routes (the same product type, and A and Z locations).
  * Identify circuits that are underused (utilization percentage > 20%).
  * Identify out of term contracts but are still being billed.


## 📊 Dataframe 'cs' Overview
The dataset is simulated using python and ChatGPT.

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


## ⚙️ Methodology
**Tools Used:**
* Python for data preprocessing and exploratory data analysis
* Power BI for visualization

### Data Preprocessing
* No empty values in columns circuit_id, a_end, z_end, product_type, supplier, start_date, monthly_recurring_cost, contract_termn_months, decom_status, and utilization_pct. Values are also in the smae format. Note that circuit_id doesn't have to be unique.
* The categories for column billing_status is shown as: 'ACTIVE BILLING', 'BILLING', 'Not Billed', 'billing'. Categories 'ACTIVE BILLING', 'BILLING', and , 'billing' are the same in meaning. The categories can be narrowed down into: 'BILLING' and 'NOT BILLED'. I created a new column called clean_billing_status consisting the two categories and removed the billing_status column.


<img width="1424" height="269" alt="image" src="https://github.com/user-attachments/assets/484d8c4b-67a9-4a39-9761-93320b94dd9d" />


 <img width="1431" height="78" alt="image" src="https://github.com/user-attachments/assets/bfba884d-1b55-4e19-99a3-63e2ec30de3b" />



|index|circuit\_id|monthly\_recurring\_cost|a\_end|z\_end|product\_type|supplier|start\_date|end\_date|contract\_term\_months|decom\_status|service\_status|reclaim|reclaim\_total|utilization\_pct|clean\_billing\_status|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|CKT-07237|2824|Dallas DC1|London DC2|Internet DIA|Orange|13-11-22|12-11-27|36|PENDING DECOM|Provisioning|NaN|NaN|34|BILLING|
|1|CKT-07085|350|Frankfurt DC1|Zurich DC1|Fiber|BT|2020-01-18|16-01-25|24|DECOM|Provisioning|NaN|NaN|75|NOT BILLED|
|2|CKT-02822|1316|London DC1|Tokyo DC1|Wave|Orange|2021/11/16|2024-11-15|36|DECOM|Active|NaN|NaN|55|BILLING|
|3|CKT-07997|4564|Zurich DC1|Amsterdam DC1|Metro Fiber|BT|28-01-21|NaN|36|ACTIVE|Pending Disconnect|NaN|NaN|94|BILLING|
|4|CKT-03339|150|Tokyo DC1|London DC1|Cross Connect|Lumen|2021/09/24|23-09-26|60|PENDING DECOM|Provisioning|NaN|NaN|61|BILLING|


* A reclaim or credit note can be expected if the circuit was reqeusted for termination, the vendor acknowledged request, yet we are still being billed. customer services that are inactive, decommissioned inventory, yet still being billed by the vendor. Upon checking, the columns reclaim and reclaim_total are logical.


 <img width="1419" height="151" alt="image" src="https://github.com/user-attachments/assets/30bb04c2-95c0-42ab-94b9-038b4ad017dd" />


|index|circuit\_id|decom\_status|service\_status|clean\_billing\_status|
|---|---|---|---|---|
|44|CKT-00705|DECOM|Inactive|BILLING|
|84|CKT-09044|DECOM|Inactive|BILLING|
|194|CKT-07283|DECOM|Inactive|BILLING|
|314|CKT-08592|DECOM|Inactive|BILLING|
|327|CKT-08965|DECOM|Inactive|BILLING|


* The categories in service_status column are: 'Active', 'Inactive', 'Pending Disconnect','Provisioning', 'Suspended', 'active'. 'Active' and 'active' are the same. To fix this, I formatted the categories to uppercase:


 <img width="1438" height="294" alt="image" src="https://github.com/user-attachments/assets/adba33d4-54e0-4b8b-b609-ccbc3ec336ea" />


* It is notable that the columns start_date and end_date are not uniform in format. There are also empty cells in the end_date. We can fill these in by adding the contract term in months to the start date.


 <img width="1438" height="131" alt="image" src="https://github.com/user-attachments/assets/dd8acf36-d3bf-4423-8031-dddba87599bc" />


|index|start\_date|end\_date|
|---|---|---|
|0|2022-11-13 00:00:00|2027-12-11 00:00:00|
|1|2020-01-18 00:00:00|2025-01-16 00:00:00|
|2|2021-11-16 00:00:00|2024-11-15 00:00:00|
|3|2021-01-28 00:00:00|NaT|
|4|2021-09-24 00:00:00|2026-09-23 00:00:00|

* Filling in empty end_date by adding the contract terms in monrth to the start date.


 <img width="1435" height="188" alt="image" src="https://github.com/user-attachments/assets/d4b6ff48-e683-49ea-9ad2-5c2444a35b88" />

  

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

### Cost-saving Objective I: Identify circuits that were requested for termination but are still being billed (services that are inactive, decommissioned, but are still being billed)

Flagging rows that are cost savings (decom_status = DECOM, billing_status = BILLING, service_status = INACTIVE


<img width="1436" height="96" alt="image" src="https://github.com/user-attachments/assets/55c73db7-870d-42d9-97dc-e0f866deab9f" />


<img width="1443" height="212" alt="image" src="https://github.com/user-attachments/assets/934c4f10-24f9-4e1a-8f37-3f3cc3100f7a" />




|index|circuit\_id|monthly\_recurring\_cost|a\_end|z\_end|product\_type|supplier|start\_date|end\_date|contract\_term\_months|decom\_status|service\_status|reclaim|reclaim\_total|utilization\_pct|clean\_billing\_status|cost\_saving|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|44|CKT-00705|2565|Singapore DC1|London DC1|Wave|Verizon|2017-12-02 00:00:00|2019-02-12 00:00:00|36|DECOM|INACTIVE|YES|30780\.0|49|BILLING|YES|
|76|CKT-07940|280|Zurich DC1|Frankfurt DC2|Fiber|AT&T|2017-12-23 00:00:00|2022-12-22 00:00:00|36|DECOM|INACTIVE|NaN|NaN|19|BILLING|YES|
|79|CKT-09319|150|Singapore DC1|Tokyo DC1|Dark Fiber|Lumen|2018-09-03 00:00:00|2021-09-03 00:00:00|36|DECOM|INACTIVE|NaN|NaN|93|BILLING|YES|
|84|CKT-09044|300|Ashburn DC1|New York DC1|Fiber|BT|2019-11-28 00:00:00|2022-11-28 00:00:00|36|DECOM|INACTIVE|YES|12957\.0|79|BILLING|YES|
|93|CKT-05688|350|Frankfurt DC2|Frankfurt DC1|Dark Fiber|BT|2018-09-12 00:00:00|2020-09-12 00:00:00|24|DECOM|INACTIVE|NaN|NaN|59|BILLING|YES|




### Cost-saving Objective II: identify circuits that are potentially duplicate routes (the same product type, and A and Z locations)

decom_status = ACTIVE, clean_billing_status = BILLING, service_status = ACTIVE and PROVISIONING)


<img width="1422" height="471" alt="image" src="https://github.com/user-attachments/assets/9b2b7cd3-8c72-4c8a-bd94-b077123dd39b" />


|index|circuit\_id|a\_end|z\_end|product\_type|duplicate\_route\_flag|
|---|---|---|---|---|---|
|545|CKT-07155|Amsterdam DC1|Frankfurt DC1|Internet DIA|true|
|1450|CKT-01831|Amsterdam DC1|Frankfurt DC1|Internet DIA|true|
|1495|CKT-02331|Amsterdam DC1|Frankfurt DC1|Internet DIA|true|
|1520|CKT-07093|Amsterdam DC1|Frankfurt DC1|Internet DIA|true|
|788|CKT-00486|Ashburn DC1|Singapore DC1|Cross Connect|true|



### Cost-saving Objective III: identify circuits that are underused (utilization percentage > 20%)

Flagging underused circuits with less than 20% utilization rate


<img width="1436" height="129" alt="image" src="https://github.com/user-attachments/assets/93a11834-8d1d-422b-adae-6f1b7ae9bc4e" />


### Cost-saving Objective IV: identify out of term contracts but are still being billed to reassess usability.


<img width="859" height="472" alt="image" src="https://github.com/user-attachments/assets/2b8717ef-813d-4028-9a64-0afd305cc73f" />


Upon closer look, inconsistencies are detected in the end_date and contract_term_months columns.
As an example, circuit CKT-07237 with both start and end dates value from the start, started on 2022-11-13. With a 36 contract term in months, it should have ended on 2025-11-13, but it shows 2027-12-11 instead.


|index|circuit\_id|start\_date|end\_date|
|---|---|---|---|
|0|CKT-07237|2022-11-13 00:00:00|2027-12-11 00:00:00|


Further investigation on contract terms is highly recommended.

## 💡 Insights
The project generated a 1.5M USD of cost savings, and consequently reclaimd 143K USD. 

The top three suppliers where we cut the most costs are BT, Orange, and Telstra. This would imply a call to closely monitor our termination requests with those suppliers.

The top three products that we have spent the most are Wave, DIA, and Metro Fiber. This could be interesting for the procurement team.

<img width="1417" height="797" alt="image" src="https://github.com/user-attachments/assets/bc907b4c-860a-4db8-90fa-53b99904de03" />

<img width="664" height="543" alt="image" src="https://github.com/user-attachments/assets/644a480d-bc8b-4364-97bc-c2e7502a286e" />


## 📈 Recommendations

* Improve internal documentation of circuit termination requests by using a project managenent tool like Jira.
* Automate cease orders internally by using ERP or CRM tool to alert finance team regarding circuits that should no longer be paid.
* Routine data monitoring and data cleaning in the database.
* Notify the network engineer and procurement teams of the underutilized and possibly duplicate circuits for reassessment.
* Conduct another project for a comprehensive data cleaning of our inventory by gathering data from our vendors, most importantly contract terms, start date, and end dates to crosscheck with our existing data.
