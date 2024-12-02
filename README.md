# Walmart E-commerce Sales Data Pipeline

This project focuses on creating a data pipeline for Walmart's e-commerce sales, analyzing the impact of public holidays on sales, and performing monthly sales trend analysis. The pipeline extracts, transforms, aggregates, and loads the data for further analysis.

## Objective

The purpose of this project is to:

- Create an automated data pipeline for merging Walmart's grocery sales data with additional data.
- Perform data cleaning and transformation to prepare the data for analysis.
- Analyze the monthly average sales trends.
- Save the cleaned and aggregated data for future insights.

## Dataset

This project uses two datasets:
1. **grocery_sales** (PostgreSQL table): Contains weekly sales data for Walmart stores with the following columns:
   - `Store_ID`: Unique identifier for the store.
   - `Date`: Date of sale.
   - `Weekly_Sales`: Sales value for the given week.

2. **extra_data.parquet** (Parquet file): Contains complementary data with the following columns:
   - `IsHoliday`: Indicates if the week contains a public holiday.
   - `Temperature`: Temperature on the day of sale.
   - `Fuel_Price`: Price of fuel in the region.
   - `CPI`: Consumer price index.
   - `Unemployment`: Unemployment rate.
   - `MarkDown1`, `MarkDown2`, `MarkDown3`, `MarkDown4`: Promotional markdowns.
   - `Dept`: Department number.
   - `Size`: Store size.
   - `Type`: Store type.

## Data Pipeline Process

The pipeline consists of four main steps: **Extract**, **Transform**, **Aggregate**, and **Load**.

### 1. **Extract**

Extract data from `grocery_sales` (SQL) and merge it with the `extra_data.parquet` file.

```python
def extract(store_data, extra_data):
    extra_df = pd.read_parquet(extra_data)
    merged_df = store_data.merge(extra_df, on="index")
    return merged_df
```
**2. Transformation**

In the transform() function, the data undergoes cleaning and transformation:

- Missing values are filled with the mean of their respective columns (e.g., `Weekly_Sales`, `CPI`, `Unemployment`).
- The `Date` column is converted to a datetime format, and the Month is extracted.
- Rows with sales lower than $10,000 are filtered out, and unnecessary columns are dropped.

``` python
def transform(raw_data):
    raw_data.fillna(
      {
          'CPI': raw_data['CPI'].mean(),
          'Weekly_Sales': raw_data['Weekly_Sales'].mean(),
          'Unemployment': raw_data['Unemployment'].mean(),
      }, inplace=True
    )
    raw_data["Date"] = pd.to_datetime(raw_data["Date"], format="%Y-%m-%d")
    raw_data["Month"] = raw_data["Date"].dt.month
    raw_data = raw_data.loc[raw_data["Weekly_Sales"] > 10000, :]
    raw_data = raw_data.drop(["index", "Temperature", "Fuel_Price", "MarkDown1", "MarkDown2", "MarkDown3", "MarkDown4", "Type", "Size", "Date"], axis=1)
    return raw_data
```
**3. Aggregate**

Calculate average weekly sales per month.

```python
def avg_weekly_sales_per_month(clean_data):
    holidays_sales = clean_data[["Month", "Weekly_Sales"]]
    holidays_sales = (holidays_sales.groupby("Month")
                      .agg(Avg_Sales=("Weekly_Sales", "mean"))
                      .reset_index().round(2))
    return holidays_sales
```
**4. Load**

Save the cleaned and aggregated data to CSV files.

```python
def load(full_data, full_data_file_path, agg_data, agg_data_file_path):
    full_data.to_csv(full_data_file_path, index=False)
    agg_data.to_csv(agg_data_file_path, index=False)
```

**5. Validation**

Ensure the data is saved correctly by checking the existence of the files.

```python
import os

def validation(file_path):
    file_exists = os.path.exists(file_path)
    if not file_exists:
        raise Exception(f"There is no file at the path {file_path}")
```

## SQL Queries

To retrieve the grocery_sales data from the PostgreSQL database:

``` sql
SELECT * FROM grocery_sales
```

## Tools and Technologies
- **Database**: PostgreSQL
- **Programming Language**: Python
- **Libraries**:
    - pandas for data manipulation
    - os for file validation

## Conclusion
A complete data pipeline has been built to analyze Walmart’s sales data. The pipeline:
- Merges the weekly sales data with complementary data.
- Performs data cleaning and transformation.
- Analyzes monthly sales trends.
- Saves the cleaned and aggregated data for further insights.
This project provides valuable insights into the relationship between public holidays and Walmart's e-commerce sales trends, with the potential for further analysis and model building.
