# Data-Engineering-Walmart-E-commerce-Analysis-Pipeline

This project builds a data pipeline to analyze Walmart's e-commerce sales data, focusing on the impact of public holidays on sales performance. It includes extracting, transforming, and loading data from multiple sources and performing data analysis on the sales trends during key holidays.

## Objective

The goal of this project is to:

- Create an end-to-end data pipeline to merge Walmart’s grocery sales data with additional data sources.
- Transform the data to clean, aggregate, and analyze the relationship between sales and holidays.
- Perform data analysis to calculate monthly sales averages and save the cleaned data for further analysis.

## Data Sources
**1. grocery_sales (PostgreSQL database):**

- Contains historical sales data for Walmart’s grocery stores.
- **Columns**: Store_ID, Date, Weekly_Sales.

**2. extra_data.parquet (parquet file):**

- Contains complementary data, including public holiday indicators, temperature, fuel price, CPI, unemployment rate, and store size/type.
- **Columns**: IsHoliday, Temperature, Fuel_Price, CPI, Unemployment, Dept, Size, Type.

## Skills and Tools Used
- **SQL**: Extracted relevant sales data using SQL queries and combined it with other data.
- **Pandas**: Used for data cleaning, merging, and transformation.
- **Parquet**: Handled the complementary data in Parquet format.
- **Matplotlib, Seaborn**: For visualizing sales trends and insights.
- **Python**: Implemented the entire data pipeline from extraction to loading.

## Data Pipeline Process
The data pipeline consists of the following steps:

**1. Extraction**
The first step involves extracting data from the grocery_sales table and merging it with the **extra_data.parquet** file.
``` python
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
