# Data Profiler

A Jupyter notebook that walks through a customer dataset end-to-end: pulling data from multiple sources, cleaning it, exploring it visually, and generating an automated profiling report.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Dataset Information](#dataset-information)
- [Project Workflow](#project-workflow)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Part A](#part-a)
- [Demo Video](#demo-video)
- [Part B: Data Acquisition](#part-b-data-acquisition)
- [Part C: Data Understanding & Cleaning](#part-c-data-understanding--cleaning)
- [Part D: Exploratory Data Analysis (EDA)](#part-d-exploratory-data-analysis-eda)
- [Part E: Data Profiling](#part-e-data-profiling)
- [Usage](#usage)
- [Output](#output)

---

## Project Overview

This project takes a raw customer dataset, brings it together from several different data sources, cleans it up, explores it visually, and finishes with an automated profiling report. It's built as a single Jupyter notebook (`data_profiler.ipynb`) split into clearly labeled parts (A–E), making it easy to follow each stage of a typical data analysis pipeline from raw ingestion to insight.

## Objectives

- Practice pulling the same data from multiple source types (CSV, JSON, SQL, API)
- Understand and clean a real-world-style dataset (missing values, duplicates, irrelevant columns)
- Explore relationships in the data through univariate, bivariate, and multivariate analysis
- Generate an automated, shareable data profiling report

## Features

- **Multi-source ingestion** — load the same data from CSV, JSON, a SQLite database, and a live REST API
- **Data cleaning** — handles missing values, checks data types, removes irrelevant columns
- **Rich visual EDA** — histograms, boxplots, a correlation heatmap, and a pair plot
- **Automated profiling report** — one-line report generation with Sweetviz, exported as HTML
- **Churn-focused analysis** — several plots and the profiling report are built around the `Churn` target column

## Technologies Used

| Category            | Tools / Libraries                     |
|---------------------|----------------------------------------|
| Language             | Python 3                              |
| Data handling        | pandas, numpy                         |
| Visualization         | matplotlib, seaborn                   |
| Database              | sqlite3                               |
| API requests           | requests                              |
| Automated profiling    | sweetviz                              |
| Environment            | Jupyter Notebook                      |

## Dataset Information

Customer-level data with the following fields:

| Column        | Description                          |
|---------------|---------------------------------------|
| `Customer_ID` | Unique identifier for each customer   |
| `Name`        | Customer name                         |
| `Age`         | Customer age                          |
| `Gender`      | Customer gender                       |
| `City`        | Customer's city                       |
| `Income`      | Customer income                       |
| `Purchases`   | Number of purchases made              |
| `Membership`  | Membership tier (Silver/Gold/Platinum)|
| `Churn`       | Whether the customer churned (Yes/No) |

## Project Workflow

- **Part A** — see [Part A](#part-a) below
- **Part B: Data Acquisition** — load the dataset from CSV, JSON, SQL, and an API
- **Part C: Data Understanding & Cleaning** — explore, handle missing values, fix types, drop irrelevant columns
- **Part D: Exploratory Data Analysis (EDA)** — univariate, bivariate, and multivariate analysis
- **Part E: Data Profiling** — generate an automated Sweetviz report

## Project Structure

```
data-profiler/
├── data_profiler.ipynb          # Main notebook (Parts B–E)
├── customer_data.csv            # Source CSV data
├── customer.json                # Source JSON data
├── customers.db                 # SQLite DB (auto-generated on run)
├── Customer_Sweetviz_Report.html # Generated profiling report (auto-generated on run)
└── README.md                    # Project documentation
```

## Requirements

```bash
pip install pandas numpy matplotlib seaborn requests sweetviz
```

Files needed in the same directory as the notebook:
- `customer_data.csv`
- `customer.json`

(`customers.db` is created automatically by the notebook when it writes the CSV data into SQLite.)

---

## Part A

*[https://drive.google.com/file/d/1mquXTRj-FDrpMp94XFA_8wuAGECeWrGb/view?usp=sharing]*

## Demo Video

*[https://drive.google.com/file/d/1THXdgkNR08WIVXORj7fryXgg3_3ooDoz/view?usp=sharing]*

```md
[![Watch the demo](thumbnail.png)](https://your-video-link-here)
```

---

## Part B: Data Acquisition

### 1. Import Libraries

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import sqlite3
import requests
import sweetviz as sv
```
Loads every library the notebook needs: `pandas`/`numpy` for data handling, `matplotlib`/`seaborn` for plotting, `sqlite3` for a local database, `requests` for API calls, and `sweetviz` for automated profiling.

### 2. Load CSV

```python
df = pd.read_csv("customer_data.csv")
print(df.head())
```
Reads the main dataset from a CSV file into a DataFrame `df` and prints the first 5 rows to confirm it loaded correctly. `df` is the DataFrame used throughout the rest of the notebook.

### 3. Load JSON

```python
json_df = pd.read_json("customer.json")
print(json_df)
```
Loads the same customer data from a JSON file into a separate DataFrame (`json_df`), demonstrating that pandas can ingest JSON just as easily as CSV.

### 4. Load SQL Table

```python
conn = sqlite3.connect("customers.db")
df.to_sql("customers", conn, if_exists="replace", index=False)
sql_df = pd.read_sql("SELECT * FROM customers", conn)
print(sql_df.head())
conn.close()
```
Connects to (or creates) a SQLite database file, writes `df` into it as a table called `customers`, then reads that table back into a new DataFrame (`sql_df`) using a SQL query. Shows the round trip: DataFrame → SQL table → DataFrame.

### 5. Load API

```python
url = "https://jsonplaceholder.typicode.com/users"
response = requests.get(url)
api_data = response.json()
api_df = pd.DataFrame(api_data)
print(api_df.head())
```
Calls a public REST API (`jsonplaceholder.typicode.com`), parses the JSON response, and converts it into a DataFrame (`api_df`) — showing how to pull data from a live web source instead of a local file.

---

## Part C: Data Understanding & Cleaning

### 6. Perform Initial Exploration

**A. `head()`, `info()`, `describe()`**

```python
## head()
print("================ Head ==================")
print(df.head())

## info()
print("================ Info ==================")
print(df.info())

## describe()
print("================ Describe ==================")
print(df.describe())
```
- `head()` — previews the first 5 rows.
- `info()` — shows column names, data types, and non-null counts (a quick way to spot missing data).
- `describe()` — gives summary statistics (mean, std, min, max, quartiles) for numeric columns.

**B. Missing Values & Duplicate Rows**

```python
## Missing values
print("=================== Missing values ===================")
print(df.isnull().sum())

## Duplicate Rows
print("=================== Duplicate Rows =====================")
print(df.duplicated().sum())
```
Counts missing (`NaN`) values per column and counts fully duplicated rows — the standard first checks before cleaning any dataset.

### 7. Apply Data Cleaning

**A. Handle Missing Data**

```python
## Handle Missing data
df["Age"] = df["Age"].fillna(df["Age"].mean())
df["Income"] = df["Income"].fillna(df["Income"].mean())
```
Fills missing values in `Age` and `Income` with the mean of each respective column, so no rows need to be dropped.

**B. Data Types**

```python
print(df.dtypes)
```
Prints the data type of every column (e.g., `int64`, `float64`, `object`) to confirm each field is stored appropriately after cleaning.

**C. Remove Irrelevant Column**

```python
df = df.drop(columns=["Customer_ID"])
```
Drops `Customer_ID` since it's just a row identifier and carries no analytical/predictive value for the EDA or profiling steps that follow.

---

## Part D: Exploratory Data Analysis (EDA)

### 8. Univariate Analysis

**A. Age / Income / Purchases Distribution**

```python
print("======================= Age Distribution =======================")
plt.figure(figsize=(6,4))
sns.histplot(df["Age"], bins=8, kde=True)
plt.title("Age Distribution")
plt.show()

## Income Distribution
print("======================= Income Distribution =======================")
plt.figure(figsize=(6,4))
sns.histplot(df["Income"], bins=8, kde=True)
plt.title("Income Distribution")
plt.show()

## Purchases Distribution
print("======================= Purchases Distribution =======================")
plt.figure(figsize=(6,4))
sns.histplot(df["Purchases"], bins=8, kde=True)
plt.title("Purchases Distribution")
plt.show()
```
Plots a histogram (with a KDE curve overlaid) for each of the three key numeric columns — `Age`, `Income`, and `Purchases` — to visualize how each variable is individually distributed (shape, spread, skew).

### 9. Bivariate Analysis

```python
## Gender vs Purchases
print("======================= Gender vs Purchases =======================")
plt.figure(figsize=(6,4))
sns.boxplot(x="Gender", y="Purchases", data=df)
plt.title("Gender vs Purchases")
plt.show()

## Income vs Churn
print("======================= Income vs Churn =======================")
plt.figure(figsize=(6,4))
sns.boxplot(x="Churn", y="Income", data=df)
plt.title("Income vs Churn")
plt.show()
```
Two boxplots comparing a numeric variable across categories: `Purchases` grouped by `Gender`, and `Income` grouped by `Churn` — useful for spotting whether these categories relate to different spending/income patterns.

### 10. Multivariate Analysis

```python
##  Correlation Heatmap
print("======================= Correlation Heatmap =======================")
plt.figure(figsize=(6,5))
corr = df.select_dtypes(include=np.number).corr()
sns.heatmap(corr, annot=True, cmap="coolwarm")
plt.title("Correlation Heatmap")
plt.show()

# Pair Plot
print("======================= Pair Plot =======================")
sns.pairplot(df, hue="Gender")
plt.show()
```
- **Correlation heatmap** — computes pairwise correlation between all numeric columns and visualizes it as an annotated heatmap, showing which variables move together.
- **Pair plot** — plots every numeric column against every other one (scatter plots + histograms), colored by `Gender`, to spot multi-variable patterns and clustering at a glance.

---

## Part E: Data Profiling

```python
report = sv.analyze(df, target_feat="Churn")
report.show_html("Customer_Sweetviz_Report.html")
```
Uses **Sweetviz** to automatically generate a full profiling report of `df` — including per-column stats, distributions, missing-value summaries, and associations — with `Churn` set as the target feature for comparison. The report is saved as `Customer_Sweetviz_Report.html`, which can be opened directly in a browser.

---

## Usage

1. Install the requirements listed above.
2. Place `customer_data.csv` and `customer.json` alongside the notebook.
3. Run all cells in order (Part B → Part E).
4. Open `Customer_Sweetviz_Report.html` in a browser to view the generated profiling report.

## Output

- Console printouts of data previews, info, and summary stats at each stage
- Histograms, boxplots, a correlation heatmap, and a pair plot
- A standalone HTML profiling report (`Customer_Sweetviz_Report.html`) generated by Sweetviz
