# REAL-ESTATES-SALES
analysis of Connecticut real estate transactions from 2001 to 2020 
REAL ESTATES SALES
OVERVIEW
The Real Estate Sales dataset, sourced from Kaggle, contains over 997,000 property transaction records from Connecticut spanning the years 2001 to 2020. It includes key details such as sale amount, assessed value, property type, town, and sale date. After cleaning with Python—standardizing column names, converting dates, handling missing values, and removing outliers—the dataset was reduced to 91,007 high-quality entries. Additional features like sales ratio and sale year were engineered to support deeper analysis. The cleaned data was exported to CSV and explored using SQL to uncover trends in sale prices over time, top-selling towns, high-value areas, property type distributions, and sales ratio patterns, forming the foundation for a dynamic Power BI dashboard.
Tools: Python, SQL, Power BI
DATA PREPARATION AND CLEANING WITH PYTHON
import pandas as pd
df = pd.read_csv("C:/Users/BEMORA/Downloads/archive (2)/Real_Estate_Sales_2001-2020_GL.csv")
print(df.head())
output:
              1
1          20002  ...                 0
2         200212  ...                 1
3         200243  ...                 1
4         200377  ...                 1

[5 rows x 11 columns]
print(df.columns)
Index(['Serial Number', 'List Year', 'Date Recorded', 'Town', 'Address',
       'Assessed Value', 'Sale Amount', 'Sales Ratio', 'Property Type',
       'Residential Type', 'Years until sold'],
      dtype='object')
df.info()
output:
 
 Serial Number  List Year Date Recorded     Town  ... Sales Ratio  Property Type  Residential Type  Years until sold
0        2020348       2020     9/13/2021  Ansonia  ...      0.4630     Commercial               Nan                 1
1          20002       2020     10/2/2020  Ashford  ...      0.5883    Residential     Single Family                 0
2         200212       2020      3/9/2021     Avon  ...      0.7248    Residential             Condo                 1
3         200243       2020     4/13/2021     Avon  ...      0.6958    Residential     Single Family                 1
4         200377       2020      7/2/2021     Avon  ...      0.5957    Residential     Single Family                 1

[5 rows x 11 columns]
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 997213 entries, 0 to 997212
Data columns (total 11 columns):
 #   Column            Non-Null Count   Dtype
---  ------            --------------   -----
 0   Serial Number     997213 non-null  int64
 1   List Year         997213 non-null  int64
 2   Date Recorded     997213 non-null  object
 3   Town              997213 non-null  object
 4   Address           997213 non-null  object
 5   Assessed Value    997213 non-null  int64
 6   Sale Amount       997213 non-null  float64
 7   Sales Ratio       997213 non-null  float64
 8   Property Type     997213 non-null  object
 9   Residential Type  997213 non-null  object
 10  Years until sold  997213 non-null  int64
dtypes: float64(2), int64(4), object(5)
memory usage: 83.7+ MB
missing = df.isnull().sum().sort_values(ascending=False)
print(missing)
output:


Serial Number       0
List Year           0
Date Recorded       0
Town                0
Address             0
Assessed Value      0
Sale Amount         0
Sales Ratio         0
Property Type       0
Residential Type    0
Years until sold    0
dtype: int64
df.columns = df.columns.str.strip().str.replace(' ', '_').str.lower()
df['date_recorded'] = pd.to_datetime(df['date_recorded'], errors='coerce')
df = df.dropna(subset=['sale_amount', 'date_recorded'])
df['assessed_value'] = df['assessed_value'].fillna(df['assessed_value'].median())
df['property_type'] = df['property_type'].fillna(df['property_type'].mode()[0])
df = df[df['sale_amount'] < df['sale_amount'].quantile(0.99)]
df['sale_year'] = df['date_recorded'].dt.year
df['sales_ratio'] = df['sale_amount'] / df['assessed_value']
df.to_csv('cleaned_real_estate_sales.csv', index=False)

EXPLORATORY DATA AMALYSIS WITH SQL
1. Total Records and Missing Value
SELECT COUNT(*) AS total_records FROM cleaned_real_estate_sales;
output:
total_records
91007

SELECT COUNT(*) AS missing_sale_amounts
FROM cleaned_real_estate_sales
WHERE sale_amount IS NULL;

output:
missing_sale_amounts
0

2. Sale Price Trends Over Time
SELECT list_year, ROUND(AVG(sale_amount), 2) AS avg_sale_price
FROM cleaned_real_estate_sales
GROUP BY list_year
ORDER BY list_year;
list_year	Avg_sale_price
2001	202583.69
2002	260955.5
2003	235386.3
2004	322407.27
2005	311718.95
2006	345666.01
2007	307958.07
2008	306685.37
2009	299977.06
2010	320552.88
2011	329439.83
2012	262542.24
2013	376009.08
2014	341679.02
2015	261818.1
2016	347933.71
2017	431081.77
2018	456932.55
2019	316602.37
2020	390898.04

3. Top Towns by Number of Sale
SELECT town, COUNT(*) AS sales_count
FROM cleaned_real_estate_sales
GROUP BY town
ORDER BY sales_count DESC
LIMIT 10;
town	Sales_count
Danbury	3290
Stamford	2724
Bristol	2662
Hartford	2502
Bridgeport	2458
Waterbury	2417
Manchester	2284
Milford	2275
Norwalk	2006
Middletown	1918

4. Highest Average Sale Prices by Town
SELECT town, ROUND(AVG(sale_amount), 2) AS avg_price
FROM cleaned_real_estate_sales
GROUP BY town
ORDER BY avg_price DESC
LIMIT 10;
town	Avg_price
New Canaan	1397888.37
Greenwich	1340225.17
Darien	1193488.82
Weston	982584.92
Wilton	913339.62
Westport	880535.71
Roxbury	853166.89
Washington	771520.13
Ridgefield	750847.04
Fairfield	728750

5. Sales Ratio Distribution
SELECT 
  CASE 
    WHEN sales_ratio < 0.5 THEN '<0.5'
    WHEN sales_ratio BETWEEN 0.5 AND 1 THEN '0.5–1.0'
    WHEN sales_ratio BETWEEN 1 AND 1.5 THEN '1.0–1.5'
    ELSE '>1.5'
  END AS ratio_range,
  COUNT(*) AS count
FROM cleaned_real_estate_sales
GROUP BY ratio_range
ORDER BY ratio_range;
Ratio_range	count
<0.5	2134
>1.5	71220
0.5–1.0	4078
1.0–1.5	13575

6. Property Type Breakdown
SELECT property_type, COUNT(*) AS count
FROM cleaned_real_estate_sales
GROUP BY property_type
ORDER BY count DESC;
Property_type	count
Residential	59859
Nan	25268
Vacant Land	3126
Commercial	1781
Apartments	397
Condo	371
Industrial	195
Public Utility	5
Two Family	2
Single Family	2
Three Family	1

DASHBOARD
 <img width="940" height="531" alt="image" src="https://github.com/user-attachments/assets/ca9b6208-37df-4d9d-8e4e-1063844487de" />





















