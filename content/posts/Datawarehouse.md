+++
date = '2025-04-01'
title = 'Datawarehouse'
+++

# A datawarehouse is a kind of database?

## No, thank you for reading.

# Jokes aside, What is a datawarehouse?

Oracle: _A data warehouse is a type of data management system that is designed to enable and support business intelligence (BI) activities, especially analytics. Data warehouses are solely intended to perform queries and analysis and often contain large amounts of historical data. The data within a data warehouse is usually derived from a wide range of sources such as application log files and transaction applications._

IBM: _A data warehouse is a system that aggregates data from multiple sources into a single, central and consistent data store. Data warehouses help prepare data for data analytics, business intelligence (BI), data mining, machine learning (ML) and artificial intelligence (AI) initiatives._

AWS: _A data warehouse is a central repository of information that can be analyzed to make more informed decisions. Data flows into a data warehouse from transactional systems, relational databases, and other sources, typically on a regular cadence. Business analysts, data engineers, data scientists, and decision makers access the data through business intelligence (BI) tools, SQL clients, and other analytics applications._

**We could say that a data warehouse is a main part of a system that handles multiple sources of data for analytical tasks**

# Before jumping into the structure, lets understand the 2 mothodologies for building a data warehouse:

all the concepts will be explained during this blog, don't worry.

### 1. **Bill Inmon** - Top Down

- The "father" of the data warehouse.
- His idea is building an enterprise data warehouse first.
- Data is integrated, consistent, and normalized.
- From that _datamarts_ are created. (a datamart is in short a small datawarehouse for a specific area of a company, like marketing, finance, sales...)

This ensures strong integration across all the company. But it takes time to implement and has a high complexity

![Inmon](https://nimbusintelligence.com/wp-content/uploads/2024/01/inmons_model-1024x532.png)

### 2. **Ralph Kimball** – Bottom-Up

- Focuses on creating _datamarts_ first, tailored to business processes.
- Datamarts are later combined into a unified Data Warehouse.
- Uses _dimensional modeling_: star and snowflake schemas.

This ensures fast results delivery sacrificing integration between domains.

Like everything in this field the approaches are both valid, there are pros and cons for each, the coice depends on your needs.

![Kimball](https://nimbusintelligence.com/wp-content/uploads/2024/01/kimballs_model-1024x511.png)

If its not clear that a data ware house is not a database here is a comparison
**OLTP systems (Online Transaction Processing)** handle the _now_, and **OLAP systems (Online Analytical Processing)** handle the _why and how over time_.

| Feature       | Traditional Database (OLTP)         | Data Warehouse (OLAP)                   |
| ------------- | ----------------------------------- | --------------------------------------- |
| Main Use      | Daily operations (CRUD)             | Business Intelligence & Analytics       |
| Data Scope    | Current, real-time data             | Historical, aggregated data             |
| Query Type    | Short, frequent transactions        | Long, complex analytical queries        |
| Schema Design | Highly normalized                   | Star or snowflake schema (denormalized) |
| Optimized For | Speed + concurrency of writes/reads | Query performance on large datasets     |
| Examples      | MySQL, PostgreSQL, MongoDB          | Redshift, BigQuery, Snowflake           |

### Star Schema:

A typical data warehouse design is the Star Schema, where a central fact table is connected to multiple dimension tables.

- Dimension tables are not normalized.
- Simple, fast queries.

```bash
             Time Dimension
                  |
     Product -- Fact Table -- Customer
                  |
              Store Dimension
```

### Snowflake Schema:

In a Snowflake Schema, dimension tables are normalized into multiple related tables.

    -Reduces data redundancy.

    -Slightly more complex queries due to more joins.

```bash
          Day ---- Time Dimension ---- Month ---- Year
                          |
     Product Category -- Product -- Fact Table -- Customer Region -- Customer
                          |
                     Store Chain -- Store
```

_(Notice how Time, Product, and Customer have extra levels now.)_

### Constellation Schema (Galaxy Schema):

In a Constellation Schema, multiple fact tables share common dimension tables.

- Suitable for complex warehouses supporting multiple processes (e.g., sales and shipments).

```bash
             Time Dimension
                  |
     Product -- Sales Fact Table -- Customer
                  |
              Store Dimension

             Time Dimension
                  |
     Product -- Shipment Fact Table -- Supplier
                  |
              Warehouse Dimension
```

_(Here, both Sales and Shipment share Product and Time dimensions.)_

### Recap:

| Schema Type   | Description                             | Pros                             | Cons               |
| ------------- | --------------------------------------- | -------------------------------- | ------------------ |
| Star          | Fact connected directly to dimensions   | Fast queries, simple design      | Some redundancy    |
| Snowflake     | Normalized dimensions                   | Less redundancy                  | More complex joins |
| Constellation | Multiple fact tables, shared dimensions | Flexibility, real-world modeling | Higher complexity  |

# Fact and Dimension Tables

## Fact Tables

Your **measurement tables**. They contain **quantitative data** that you want to analyze, such as:

- Revenue
- Sales quantity
- Number of website visits
- Profit margins

### Characteristics:

- Usually **very large**
- Contain **foreign keys** to related dimension tables
- Include **measurable data** (metrics)

```sql
| Date_ID | Product_ID | Store_ID | Sales_Amount | Quantity |
|---------|------------|----------|--------------|----------|
| 20240101| 1001       | 21       | 1200         | 3        |
```

## Dimension tables

These ones describe the **context** of your **facts**, they have **descriptive attributes** for example:

- Product details (name, category)
- Customer info (name, age group, location)
- Time (day, week, quarter)
- Store (region, city)

### Characteristics:

- Generally smaller than fact tables.
- Contain attributes used for filtering, grouping, labeling.
- Each row gives more detail to your fact table data.

### Example:

```sql
| Product_ID | Name     | Category | Brand |
| ---------- | -------- | -------- | ----- |
| 1001       | Sneakers | Footwear | Nike  |
```

### Measures vs Dimensions

#### Measure

Numerical value to be analyzed, this are found in fact tables (total sales, units sold, profit margin)

### Dimension

Descriptive attribute that provides context, found in a dimension table (product name, customer region, date)

### Differences

| Feature  | Measure                     | Dimension                       |
| -------- | --------------------------- | ------------------------------- |
| Type     | Numeric / Quantitative      | Textual / Descriptive           |
| Location | Fact Table                  | Dimension Table                 |
| Purpose  | To be aggregated (SUM, AVG) | To slice, filter, group data    |
| Examples | Sales, Cost, Quantity       | Customer Name, Product Category |

## The Time Dimension and Its Role

_essential in a data warehouse because nearly every analysis is time-based: by day, week, quarter, year, etc._

### The Surrogate Date Key

Rather than using a native DATE type directly, warehouses use a surrogate key (like Date_ID = 20240101) that links facts to dates.

Attributes Often Include:

- Full Date

- Day of Week

- Month / Year / Quarter

- Is Weekday / Weekend

- Fiscal Year indicators

This allows very efficient grouping and slicing of data over time.

## Why Fact and Dimension Tables Must Be One-to-Many

_A fact table should always have a many-to-one relationship with each dimension table._

Why Many-to-Many Doesn’t Work

- Creates ambiguity in joins: if a single fact is linked to multiple rows in a dimension, aggregate queries would return duplicated or incorrect results.

- Violates the Star Schema design principle.

- Causes data integrity issues and slows down query performance.

If there's a natural many-to-many relationship (like multiple authors per book), it's handled by bridge/junction tables.
