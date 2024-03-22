# ExerciseDBT

# Analytics Engineering Exercise with DBT, PostgreSQL, and Metabase

This comprehensive exercise guides you through setting up a data pipeline with PostgreSQL, DBT, and Metabase. You'll start by importing data, then move on to modeling the data with DBT and visualizing insights with Metabase. Refere to dbt documentation to understand the different commands : https://docs.getdbt.com/docs/build/projects.

### Summarry
#### Part 1 & 2 : Those two parts are only for initial setup, there is no delivery expected 

#### Part 3
Expected delivery:

- one model `stg_sales.sql` (based on the provided code)
- one model `stg_products.sql` (your turn to develop it, inspire yourself from the example model `stg_sales.sql`)
- unique / not_null tests in `stg/model.yml`
 

#### Part 4
Expected delivery:

- one model `int_daily_sales.sql` (based on the provided code)
- one model `int_category_sales.sql`
- unique / not_null tests in `stg/model.yml`

#### Part 5
Expected delivery:
- one model `daily_sales_volume.sql` (based on the provided code)
- one model `daily_sales_revenue_by_category.sql`


#### Part 6
Expected delivery:
A dashboard, containing at least the four charts. Provide a screenshot in you final delivery

#### Part 7
A github repository, send the link by email

Expected project structure : 

```bash
dbt_project/
├── seeds/
│ ├── raw_sales_data.csv
│ ├── raw_product_data.csv
├── models/
│ │
│ ├── staging/
│ │ ├── stg_sales.sql # Staging model for raw sales data
│ │ └── stg_products.sql # Staging model for raw product data
│ │ └── model.yml # Yaml file containing tests definitions
│ │
│ └── marts/
│   ├── daily_sales_volume.sql # Daily sales aggregation
│   └── daily_sales_revenue_by_category.sql # Daily sales aggregation by category
```



## 1. Setting Up the DBT Project

### Install Dependencies

#### Create a Virtual Environment

First, you need to create a virtual environment where you can install DBT. This keeps your project's dependencies isolated from your global Python environment.
This helps in managing dependencies and ensures that DBT runs smoothly without any version conflicts with other Python packages you might have installed.

- **For Windows Users:**

```cmd
python -m venv dbt-env
dbt-env\Scripts\activate
```

- **For Unix/macOS Users:**

```bash
python3 -m venv dbt-env
source dbt-env/bin/activate
```

These commands create a new virtual environment named dbt-env and activate it. python command might be python3 on some Windows systems where both Python 2 and Python 3 are installed.

Install dbt-core and dbt-postgres
With the virtual environment activated, install dbt-core and dbt-postgres using pip. These packages are required to run DBT with a PostgreSQL database.

```bash
pip install dbt-core dbt-postgres
```

### Configure dbt profile
Simply run `dbt init` and follow the steps.

You'll need to fill the following informations : 
- Host (use your IP address)
- database name (postgres database name)
- password (postgres database password)
- port (5432)

## 1. Preparing the Raw data

For simplicity, we'll use raw data from csv files. In the real life, data would have been a raw table.
> Ensure you've downloaded the two csv files and store them under `seeds` directory in your project.

Simply run the following command : `dbt seed`.  It should create two views :
- `raw_product_data`
- `raw_sales_data`


## 3. Creating Staging Models
### 3.1 Staging Sales Data (stg_sales.sql)

Create a file `stg_sales.sql` under `models/staging` and add the following content.

```sql
{{ config(materialized='view') }}

select
    sale_id,
    product_id,
    quantity::integer as quantity,
    sale_date::date as sale_date
from {{ ref('raw_sales_data') }}
```

Once it's done simply run the following commands `dbt run --select stg_sales`. It will create a view in postgres.

Run a `select * from stg_sales` to visualize the result.
> You can do it from metabase, pgadmin, or psql.


### 3.2 Staging Product Data (stg_products.sql)
Now create a new model `stg_products.sql` based on `raw_product_data` under `models/staging/`. Use the `stg_sales` example as a reference.

You need to : 
- cast product_name to `varchar(255)`
- cast category to `varchar(50)`d
- cast price to `numeric(10, 2)`

```sql
{{ config(materialized='view') }}

select
    product_id,
    product_name::varchar(255) as product_name,
    category::varchar(50) as category,
    price::numeric(10, 2) as price
from {{ source('raw', 'raw_product_data') }}
```

Run a `select * from stg_sales` to visualize the result.


### 3.3 Testing Staging Models
Add tests for uniqueness and not-null constraints to your staging models within a `models.yml` file.
Under `models/staging/model.yml`, add the following content:
```yml
version: 2

models:
  - name: stg_sales
    description: "A starter dbt model"
    columns:
      - name: sale_id
        description: "The primary key for this table"
        tests:
          - unique
          - not_null
```

Then run `dbt test --select stg_sales`.

Now create a similar test for `stg_products` to ensure `product_id` is unique and not_null.

Then run `dbt test --select stg_products`.

## 4. Marts Models
Create models to perform aggregations and enrich the data.

### 4.2 Example
Create `daily_sales_volume.sql` model under `models/marts/`. 

```sql
{{ config(materialized='table') }}

select
    sale_date,
    count(*) as total_sales
from {{ ref('stg_sales') }}
group by sale_date
```

> This model compute the number of sales per day.

### 4.3 Exercise

Create `daily_sales_revenue_by_category.sql` model under `models/marts/`. 
**You need to compute the daily revenue.**

To do so, you'll need to join `stg_sales`and `stg_products` using `product_id` key.

The daily revenue is defined as : `quantity * price`.


## 5. Dashboard Creation with Metabase
Create the following metabase visualization in one dashboard `sales`:

- Daily Sales Trend: A line chart showing sales over time.
- Monthly Sales Trend: A line chart showing sales by month.
- Sales by Category: A pie chart showing the revenue contribution by category.
- Category Performance Over Time: Line charts for each product category's trends.

**Those are the minimum visualization to provide, feel free to add any of your choice**


## 6. Sharing Your Work
Initialize a Git repository in your DBT project directory, commit your changes, and push to a new GitHub repository. Instructions for creating a new repository are available on GitHub's website.
